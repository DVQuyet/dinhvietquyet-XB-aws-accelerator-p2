Tiếp theo sau **GitHub Actions & CI/CD** là **ArgoCD Getting Started**.

Hiểu đơn giản: **ArgoCD là công cụ GitOps dùng để kéo YAML/manifest từ Git về Kubernetes cluster và tự đồng bộ cluster theo Git.**

Nếu GitHub Actions là “người kiểm tra code trước khi merge”, thì ArgoCD là “người canh cluster chạy đúng theo Git”.

---

# 1. ArgoCD là gì?

**ArgoCD = GitOps controller cho Kubernetes.**

Nó làm nhiệm vụ chính:

```text
Đọc manifest trong Git
→ so sánh với Kubernetes cluster
→ thấy lệch thì báo OutOfSync
→ sync để cluster giống Git
```

Ví dụ Git có file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
```

Nếu cluster thật chỉ có 1 pod, ArgoCD sẽ phát hiện:

```text
Git muốn: replicas = 3
Cluster thật: replicas = 1
Trạng thái: OutOfSync
```

Sau đó mình bấm **Sync** hoặc bật auto-sync thì ArgoCD sẽ đưa cluster về đúng 3 replicas.

---

# 2. Vì sao cần ArgoCD?

Không có ArgoCD, bạn thường deploy bằng tay:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Cách này dễ gặp lỗi:

```text
Không biết ai apply
Không biết apply bản nào
Không biết cluster có bị sửa tay không
Rollback khó hơn
Không có giao diện theo dõi trạng thái app rõ ràng
```

Có ArgoCD thì luồng sẽ là:

```text
Developer sửa YAML
→ push lên Git
→ ArgoCD đọc Git
→ ArgoCD sync vào Kubernetes
→ cluster chạy đúng theo Git
```

Nói ngắn:

```text
Không apply manifest tay nữa.
Git là nguồn chuẩn.
ArgoCD là người đồng bộ.
```

---

# 3. ArgoCD cần gì trước khi cài?

Theo tài liệu Getting Started, trước khi cài ArgoCD cần có:

```text
1. kubectl
2. kubeconfig
3. Kubernetes cluster
4. CoreDNS
```

Tài liệu ArgoCD yêu cầu có `kubectl`, có file `kubeconfig`, và CoreDNS trước khi bắt đầu cài đặt. 

Giải thích dễ hiểu:

## kubectl

`kubectl` là command-line tool để nói chuyện với Kubernetes.

Ví dụ:

```bash
kubectl get pods
kubectl get svc
kubectl apply -f app.yaml
```

Không có `kubectl` thì bạn không điều khiển được cluster.

---

## kubeconfig

`kubeconfig` là file chứa thông tin kết nối tới Kubernetes cluster.

Nó thường nằm ở:

```text
~/.kube/config
```

Trong đó có:

```text
Cluster ở đâu
User là ai
Token/certificate để đăng nhập
Context hiện tại là cluster nào
```

Nói dễ hiểu:

```text
kubectl = cái điện thoại
kubeconfig = danh bạ + chìa khóa để gọi đúng cluster
```

---

## Kubernetes cluster

Ở W8 bạn đã có minikube cluster, nên W9 dùng cluster đó để GitOps hóa.

Ví dụ kiểm tra cluster:

```bash
kubectl cluster-info
kubectl get nodes
```

Nếu thấy node chạy là ổn.

---

# 4. Bước 1 — Cài ArgoCD vào cluster

Lệnh chính:

```bash
kubectl create namespace argocd
```

Lệnh này tạo namespace riêng cho ArgoCD.

Namespace hiểu đơn giản là “khu vực riêng” trong Kubernetes.

Sau đó cài ArgoCD:

```bash
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Tài liệu nói lệnh này sẽ tạo namespace `argocd`, nơi các service và application resource của ArgoCD sẽ nằm, rồi apply manifest chính thức để cài ArgoCD. 

Sau khi chạy, trong cluster sẽ có nhiều component của ArgoCD, ví dụ:

```text
argocd-server
argocd-repo-server
argocd-application-controller
argocd-dex-server
argocd-redis
```

Bạn có thể kiểm tra:

```bash
kubectl get pods -n argocd
```

Khi pod chạy ổn, sẽ thấy dạng:

```text
NAME                                  READY   STATUS
argocd-server-xxx                     1/1     Running
argocd-repo-server-xxx                1/1     Running
argocd-application-controller-xxx     1/1     Running
```

---

# 5. `--server-side --force-conflicts` là gì?

Trong lệnh cài có đoạn:

```bash
--server-side --force-conflicts
```

Hiểu đơn giản:

## `--server-side`

Thay vì kubectl xử lý apply ở máy bạn, Kubernetes API server xử lý việc apply.

Tài liệu giải thích flag này cần thiết vì một số CRD của ArgoCD, như ApplicationSet, có thể vượt giới hạn annotation 262KB nếu dùng client-side apply. Server-side apply tránh giới hạn này. 

Nói dễ nhớ:

```text
--server-side = để Kubernetes server xử lý apply, tránh lỗi manifest quá lớn.
```

## `--force-conflicts`

Cho phép apply lấy quyền quản lý một số field nếu trước đó field đó do tool khác quản lý.

Tài liệu nói flag này hữu ích cho fresh install và cần thiết khi upgrade, nhưng có thể ghi đè custom modification ở các field nằm trong manifest ArgoCD. 

Nói dễ nhớ:

```text
--force-conflicts = nếu có tranh chấp field, cho manifest ArgoCD thắng.
```

---

# 6. Bước 2 — Cài ArgoCD CLI

ArgoCD có giao diện web, nhưng cũng có CLI tên là `argocd`.

Tài liệu hướng dẫn tải ArgoCD CLI từ release mới nhất hoặc cài bằng Homebrew trên Mac/Linux/WSL. 

Ví dụ nếu dùng WSL/Linux có Homebrew:

```bash
brew install argocd
```

Trên Windows, thường tải file binary từ GitHub Releases rồi thêm vào PATH.

Kiểm tra cài xong:

```bash
argocd version
```

---

# 7. Bước 3 — Truy cập ArgoCD UI

Mặc định ArgoCD **không expose ra ngoài cluster**.

Tài liệu nói mặc định ArgoCD không thể truy cập từ bên ngoài cluster, nên cần dùng một trong các cách: LoadBalancer, Ingress hoặc Port Forwarding. 

Với minikube/local, dễ nhất là dùng port-forward:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Sau đó mở trình duyệt:

```text
https://localhost:8080
```

Nếu trình duyệt báo warning certificate thì bình thường, vì bản mặc định dùng self-signed certificate.

---

# 8. Bước 4 — Lấy mật khẩu admin ban đầu

ArgoCD tạo sẵn user:

```text
username: admin
```

Mật khẩu ban đầu nằm trong secret `argocd-initial-admin-secret`.

Tài liệu nói initial password của tài khoản admin được auto-generate và lưu trong field `password` của secret `argocd-initial-admin-secret`; có thể lấy bằng CLI. 

Lệnh lấy password:

```bash
argocd admin initial-password -n argocd
```

Sau đó login:

```bash
argocd login localhost:8080
```

Nếu dùng HTTPS local có self-signed certificate, có thể cần:

```bash
argocd login localhost:8080 --insecure
```

Sau khi login, nên đổi password:

```bash
argocd account update-password
```

Tài liệu cũng khuyến nghị xóa secret initial password sau khi đã đổi mật khẩu vì secret này chỉ dùng để lưu password ban đầu. 

---

# 9. Bước 5 — Register cluster

Phần này **optional** nếu ArgoCD deploy vào chính cluster nó đang chạy.

Tài liệu nói nếu deploy vào cùng cluster với ArgoCD, dùng địa chỉ Kubernetes API server:

```text
https://kubernetes.default.svc
```

Còn nếu deploy sang cluster bên ngoài thì mới cần register cluster. 

Ví dụ xem context hiện có:

```bash
kubectl config get-contexts -o name
```

Nếu muốn add cluster ngoài:

```bash
argocd cluster add docker-desktop
```

Nhưng với W9 dùng minikube cùng cluster, bạn thường dùng:

```text
https://kubernetes.default.svc
```

Nói dễ hiểu:

```text
ArgoCD ở trong minikube
App cũng deploy vào minikube
=> không cần add external cluster
```

---

# 10. Bước 6 — Tạo Application từ Git repo

Đây là phần quan trọng nhất.

Trong ArgoCD, **Application** là object nói cho ArgoCD biết:

```text
Repo Git nào?
Path nào trong repo?
Deploy vào cluster nào?
Deploy vào namespace nào?
Có auto sync không?
```

Ví dụ lệnh tạo app guestbook:

```bash
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Tài liệu dùng repo `argocd-example-apps` và app `guestbook` để demo cách ArgoCD hoạt động. 

Giải thích:

```text
guestbook
→ tên Application trong ArgoCD

--repo
→ Git repo chứa manifest

--path
→ thư mục trong repo chứa YAML

--dest-server
→ cluster đích

--dest-namespace
→ namespace đích
```

Nói đơn giản:

```text
ArgoCD Application = bản khai báo “hãy lấy YAML ở repo/path này và deploy vào cluster/namespace này”.
```

---

# 11. Ví dụ Application YAML

Thay vì tạo bằng CLI, bạn có thể viết manifest:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/your-name/your-repo.git
    targetRevision: HEAD
    path: cloud/w9/day-a/k8s

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Giải thích:

```text
kind: Application
→ đây là app do ArgoCD quản lý

source.repoURL
→ repo Git chứa manifest

source.path
→ thư mục chứa YAML

destination.server
→ Kubernetes cluster đích

destination.namespace
→ namespace deploy

syncPolicy.automated
→ bật tự sync

prune: true
→ Git xóa resource thì cluster cũng xóa theo

selfHeal: true
→ ai sửa tay trong cluster thì ArgoCD kéo lại đúng Git
```

---

# 12. Bước 7 — Sync Application

Sau khi tạo app, ban đầu app có thể ở trạng thái:

```text
OutOfSync
Missing
```

Tài liệu nói app guestbook ban đầu là `OutOfSync` vì app chưa được deploy và Kubernetes resource chưa được tạo. 

Kiểm tra app:

```bash
argocd app get guestbook
```

Sync app:

```bash
argocd app sync guestbook
```

Tài liệu nói lệnh sync sẽ lấy manifest từ repository và thực hiện apply manifest vào cluster. 

Sau khi sync xong:

```text
Sync Status: Synced
Health Status: Healthy
```

---

# 13. Các trạng thái quan trọng trong ArgoCD

## Synced

```text
Git và cluster giống nhau.
```

Ví dụ:

```text
Git muốn Deployment web replicas=3
Cluster đang replicas=3
=> Synced
```

---

## OutOfSync

```text
Git và cluster đang lệch nhau.
```

Ví dụ:

```text
Git muốn image web:v2
Cluster đang image web:v1
=> OutOfSync
```

---

## Healthy

```text
Resource chạy ổn.
```

Ví dụ pod Running, deployment đủ replicas.

---

## Degraded

```text
Resource có vấn đề.
```

Ví dụ pod CrashLoopBackOff, deployment không đủ replicas.

---

## Missing

```text
Git có khai báo resource nhưng cluster chưa có resource đó.
```

Ví dụ Git có Deployment `web`, nhưng cluster chưa tạo Deployment đó.

---

# 14. Luồng ArgoCD hoạt động đầy đủ

Bạn nên học thuộc luồng này:

```text
1. Bạn viết manifest Kubernetes trong Git
2. Bạn tạo ArgoCD Application trỏ vào repo/path đó
3. ArgoCD đọc Git
4. ArgoCD so sánh Git với cluster
5. Nếu lệch → OutOfSync
6. Bạn bấm Sync hoặc bật Auto Sync
7. ArgoCD apply manifest vào cluster
8. Cluster chạy đúng như Git
9. Nếu ai sửa tay trong cluster → ArgoCD phát hiện drift
10. Nếu bật selfHeal → ArgoCD tự sửa lại theo Git
```

---

# 15. ArgoCD khác gì `kubectl apply`?

`kubectl apply`:

```text
Bạn tự chạy lệnh
Chạy xong là thôi
Không tự canh drift
Không có dashboard GitOps
Không tự biết Git là nguồn chuẩn
```

ArgoCD:

```text
Tự đọc Git
Tự so sánh với cluster
Có UI xem resource tree
Báo Synced/OutOfSync/Healthy/Degraded
Có auto-sync/self-heal/prune
```

Bảng dễ nhớ:

| Nội dung          | kubectl apply                  | ArgoCD                 |
| ----------------- | ------------------------------ | ---------------------- |
| Ai deploy?        | Người chạy lệnh                | ArgoCD controller      |
| Nguồn chuẩn       | File local hoặc repo tùy người | Git repo               |
| Theo dõi drift    | Không tự động                  | Có                     |
| UI xem trạng thái | Không                          | Có                     |
| Auto sync         | Không                          | Có                     |
| Rollback GitOps   | Khó hơn                        | Dễ hơn qua Git/history |

---

# 16. Trong W9 day-a cần làm gì với ArgoCD?

Theo mục tiêu W9:

```text
Cluster W8 đã có giờ GitOps-managed.
Không apply manifest tay nữa.
```

Nghĩa là bạn cần chuyển app W8 sang dạng:

```text
Manifest nằm trong Git
ArgoCD Application trỏ vào manifest đó
Deploy bằng ArgoCD sync
Không deploy bằng kubectl apply thủ công
```

Cấu trúc repo gợi ý:

```text
cloud/
  w9/
    day-a/
      argocd/
        application.yaml
      k8s/
        deployment.yaml
        service.yaml
        configmap.yaml
```

Trong đó:

```text
k8s/
→ chứa manifest app

argocd/application.yaml
→ khai báo ArgoCD Application trỏ vào thư mục k8s/
```

---

# 17. Ví dụ cho repo W9 của bạn

Giả sử app của bạn nằm ở:

```text
cloud/w9/day-a/k8s/
```

Bạn tạo file:

```text
cloud/w9/day-a/argocd/application.yaml
```

Nội dung:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: w8-platform
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/YOUR_USERNAME/YOUR_REPO.git
    targetRevision: HEAD
    path: cloud/w9/day-a/k8s

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Apply Application lần đầu:

```bash
kubectl apply -f cloud/w9/day-a/argocd/application.yaml
```

Sau đó ArgoCD sẽ quản lý app.

Từ lúc này trở đi, khi muốn sửa app:

```text
Không sửa trực tiếp bằng kubectl.
Không scale tay.
Không edit deployment trong cluster.
Sửa YAML trong Git rồi commit/push.
```

---

# 18. Những lệnh cần nhớ

```bash
# Tạo namespace cho ArgoCD
kubectl create namespace argocd

# Cài ArgoCD
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Xem pod ArgoCD
kubectl get pods -n argocd

# Port-forward UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Lấy password admin ban đầu
argocd admin initial-password -n argocd

# Login CLI
argocd login localhost:8080 --insecure

# Tạo app bằng CLI
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Xem app
argocd app get guestbook

# Sync app
argocd app sync guestbook
```

---

# 19. Các từ cần thuộc

```text
ArgoCD
= GitOps controller cho Kubernetes

Application
= object khai báo repo/path/cluster/namespace để ArgoCD deploy

Sync
= đồng bộ cluster theo Git

Synced
= Git và cluster giống nhau

OutOfSync
= Git và cluster bị lệch

Healthy
= app/resource chạy ổn

Degraded
= app/resource có lỗi

Prune
= Git xóa resource thì cluster cũng xóa

Self-heal
= ai sửa tay trong cluster thì ArgoCD tự kéo lại theo Git

Destination
= cluster/namespace đích

Source
= Git repo/path chứa manifest
```

---

# 20. Chốt lại phần ArgoCD Getting Started

Bạn chỉ cần nhớ:

```text
ArgoCD là công cụ GitOps cho Kubernetes.
Nó lấy manifest từ Git, so sánh với cluster, rồi sync cluster theo Git.
```

Luồng quan trọng:

```text
Git repo chứa YAML
→ ArgoCD Application trỏ vào repo/path
→ ArgoCD phát hiện OutOfSync
→ Sync
→ Kubernetes chạy đúng theo Git
```

Cách nhớ siêu ngắn:

```text
Git = bản thiết kế
Kubernetes = công trình thật
ArgoCD = giám sát công trình, thấy sai thì sửa theo bản thiết kế
```
