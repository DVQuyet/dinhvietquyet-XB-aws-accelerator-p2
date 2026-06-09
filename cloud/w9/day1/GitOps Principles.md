Nội dung đầu tiên của ngày T2 là **GitOps Principles** — tức là “4 nguyên tắc cốt lõi của GitOps”. Hiểu đơn giản: **GitOps là cách quản lý hệ thống bằng Git**. Mình không vào server rồi sửa tay nữa, mà viết trạng thái mong muốn vào Git, sau đó tool như ArgoCD/Flux tự kéo về và làm cluster giống Git.

Nguồn chính nói GitOps có 4 nguyên tắc: **Declarative**, **Versioned and Immutable**, **Pulled Automatically**, **Continuously Reconciled**. 

---

# 1. GitOps là gì?

**GitOps = Git + Operations.**

Tức là mình dùng Git để quản lý việc vận hành hệ thống, đặc biệt là Kubernetes.

Ví dụ trước đây bạn deploy thủ công:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Cách này có vấn đề:

* Ai apply?
* Apply lúc nào?
* File nào là bản đúng?
* Nếu cluster bị sửa tay thì làm sao biết?
* Muốn rollback thì quay lại bản nào?

GitOps giải quyết bằng cách:

```text
Git repo = nơi chứa cấu hình chuẩn
Cluster = nơi chạy thật
ArgoCD/Flux = robot canh Git và cluster
```

Nếu Git ghi rằng app cần chạy **3 replicas**, nhưng cluster đang có **1 replica**, ArgoCD sẽ thấy lệch và kéo cluster về đúng **3 replicas**.

---

# 2. Desired State là gì?

**Desired State = trạng thái mong muốn.**

Hiểu như “bản thiết kế chuẩn” của hệ thống.

Ví dụ bạn muốn app chạy như sau:

```yaml
replicas: 3
image: my-app:v1
service:
  type: ClusterIP
```

Đây là **desired state**.

Còn cluster thật đang chạy cái gì thì gọi là **actual state**.

```text
Desired State: app chạy 3 pod, image v1
Actual State: app đang chạy 1 pod, image v1
```

Hai cái này đang lệch nhau.

Trong glossary, desired state được mô tả là toàn bộ dữ liệu cấu hình đủ để tái tạo lại hệ thống sao cho hệ thống chạy giống như mong muốn. 

---

# 3. Nguyên tắc 1 — Declarative

**Declarative = khai báo mình muốn kết quả cuối cùng là gì, không cần ghi từng bước làm.**

Ví dụ đời thường:

```text
Tôi muốn có 1 ly cà phê sữa đá.
```

Đó là declarative. Bạn nói kết quả mong muốn.

Còn imperative là:

```text
Lấy ly.
Cho đá.
Cho cà phê.
Cho sữa.
Khuấy lên.
```

Trong Kubernetes/GitOps, declarative nghĩa là bạn viết YAML mô tả trạng thái mong muốn:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
```

Bạn không cần nói Kubernetes phải tạo pod theo từng bước. Bạn chỉ nói: “Tôi muốn Deployment web có 3 replicas”.

Source nói hệ thống GitOps phải có desired state được biểu diễn theo cách declarative. 

**Câu dễ nhớ:**

```text
Declarative = viết cái mình muốn, không viết từng bước làm.
```

---

# 4. Nguyên tắc 2 — Versioned and Immutable

**Versioned = có lịch sử phiên bản.**
**Immutable = đã ghi rồi thì không sửa ngầm, muốn đổi thì tạo commit mới.**

Vì desired state nằm trong Git, mọi thay đổi đều có commit:

```text
commit A: replicas = 2
commit B: replicas = 3
commit C: image = v2
```

Nhờ vậy mình biết:

* Ai sửa?
* Sửa gì?
* Sửa lúc nào?
* Vì sao sửa?
* Muốn quay lại bản cũ thì quay lại commit nào?

Ví dụ bạn deploy bản lỗi `image: my-app:v2`, sau đó app chết. Nếu dùng GitOps, bạn có thể `git revert` commit đó để quay lại `image: my-app:v1`.

Source nói desired state phải được lưu theo cách có tính bất biến, có versioning và giữ đầy đủ lịch sử phiên bản. 

**Câu dễ nhớ:**

```text
Versioned and Immutable = mọi thay đổi phải qua commit, có lịch sử, dễ rollback.
```

---

# 5. Nguyên tắc 3 — Pulled Automatically

**Pulled Automatically = agent tự kéo cấu hình từ Git về.**

Đây là điểm rất quan trọng.

Với CI/CD truyền thống, thường là kiểu **push**:

```text
GitHub Actions chạy xong → đẩy lệnh deploy vào cluster
```

Còn GitOps chuẩn là kiểu **pull**:

```text
ArgoCD/Flux nằm trong cluster → tự nhìn Git → tự kéo manifest về → tự sync
```

Ví dụ:

```text
Bạn push code YAML lên Git
ArgoCD thấy Git thay đổi
ArgoCD kéo manifest mới về
ArgoCD apply vào Kubernetes
```

Glossary giải thích rằng GitOps dùng pull vì agent cần truy cập desired state từ state store bất cứ lúc nào, không chỉ khi có một sự kiện push xảy ra. 

**Vì sao pull tốt hơn push?**

Vì cluster tự chủ động kiểm tra Git. Nếu ai đó sửa tay trong cluster, ArgoCD vẫn phát hiện lệch và sửa lại, dù không có GitHub Actions nào chạy.

**Câu dễ nhớ:**

```text
Pulled Automatically = robot trong cluster tự kéo cấu hình từ Git về.
```

---

# 6. Nguyên tắc 4 — Continuously Reconciled

**Continuously Reconciled = liên tục so sánh actual state với desired state và cố gắng sửa cho khớp.**

Đây là trái tim của GitOps.

```text
Git nói: replicas = 3
Cluster thật: replicas = 1
ArgoCD thấy lệch
ArgoCD sửa cluster về replicas = 3
```

Sự lệch này gọi là **drift**.

Source nói drift là khi actual state của hệ thống đã hoặc đang lệch khỏi desired state. 

Còn **reconciliation** là quá trình đảm bảo actual state khớp với desired state. 

Ví dụ dễ hiểu:

```text
Git = đáp án đúng trên bảng
Cluster = bài làm hiện tại
ArgoCD = giáo viên đi kiểm tra liên tục

Nếu bài làm khác đáp án → sửa lại cho đúng.
```

**Câu dễ nhớ:**

```text
Continuously Reconciled = liên tục canh lệch và kéo hệ thống về đúng Git.
```

---

# 7. State Store là gì?

**State Store = nơi lưu desired state.**

Trong GitOps, state store thường là Git repository.

Ví dụ repo:

```text
cloud/
  w9/
    day-a/
      argocd/
        app.yaml
      k8s/
        deployment.yaml
        service.yaml
```

Git là nơi chứa bản chuẩn. Cluster chỉ là nơi chạy theo bản chuẩn đó.

Glossary nói Git là ví dụ kinh điển của state store vì nó lưu các phiên bản immutable của desired state và có kiểm soát/audit thay đổi. 

---

# 8. Drift là gì?

**Drift = cluster thật bị lệch khỏi Git.**

Ví dụ Git ghi:

```yaml
replicas: 3
```

Nhưng ai đó chạy tay:

```bash
kubectl scale deployment web --replicas=1
```

Lúc này:

```text
Git muốn: 3 pod
Cluster thật: 1 pod
=> Drift
```

Nếu có GitOps, ArgoCD sẽ phát hiện và có thể sửa lại thành 3 pod.

---

# 9. Reconciliation là gì?

**Reconciliation = quá trình sửa lệch.**

Ví dụ:

```text
Bước 1: ArgoCD đọc Git
Bước 2: ArgoCD đọc cluster thật
Bước 3: So sánh
Bước 4: Nếu lệch → sync lại
```

Nói ngắn:

```text
Reconciliation = so sánh + sửa cho đúng.
```

---

# 10. GitOps khác CI/CD truyền thống thế nào?

CI/CD truyền thống thường hoạt động kiểu:

```text
Code đổi → pipeline chạy → pipeline deploy vào cluster
```

GitOps hoạt động kiểu:

```text
Git đổi → ArgoCD thấy đổi → ArgoCD kéo về → cluster tự sync
```

Khác biệt quan trọng:

| Nội dung              | CI/CD truyền thống        | GitOps                         |
| --------------------- | ------------------------- | ------------------------------ |
| Ai deploy?            | Pipeline đẩy vào cluster  | Agent trong cluster kéo từ Git |
| Nguồn chuẩn           | Có thể là pipeline/script | Git repo                       |
| Sửa tay trong cluster | Có thể không ai biết      | ArgoCD phát hiện drift         |
| Rollback              | Tùy pipeline              | Quay lại commit cũ             |
| Audit                 | Có thể rời rạc            | Git ghi lịch sử rõ             |

---

# 11. Ví dụ tổng hợp dễ hiểu

Giả sử bạn có app `web`.

Trong Git:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: web
          image: web:v1
```

ArgoCD đọc Git và apply vào cluster.

Sau đó bạn muốn nâng app lên `v2`, bạn sửa Git:

```yaml
image: web:v2
```

Rồi commit:

```bash
git add .
git commit -m "[W9-D1] update web image to v2"
git push
```

ArgoCD thấy Git thay đổi và sync cluster sang `web:v2`.

Nếu `v2` lỗi, bạn rollback bằng Git:

```bash
git revert <commit-id>
git push
```

ArgoCD thấy Git quay lại `v1`, rồi sync cluster về `web:v1`.

---

# 12. Chốt lại phần đầu tiên cần thuộc

Bạn chỉ cần nhớ câu này:

```text
GitOps là cách vận hành hệ thống bằng Git.
Git lưu trạng thái mong muốn.
ArgoCD/Flux tự kéo cấu hình từ Git.
Nếu cluster lệch khỏi Git, tool sẽ tự phát hiện và kéo về đúng.
```

4 nguyên tắc phải thuộc:

```text
1. Declarative
   Viết trạng thái mong muốn bằng YAML/manifest.

2. Versioned and Immutable
   Mọi thay đổi nằm trong Git, có commit history.

3. Pulled Automatically
   Agent như ArgoCD/Flux tự kéo cấu hình từ Git.

4. Continuously Reconciled
   Agent liên tục so sánh Git với cluster và sửa lệch.
```

Cách nhớ siêu ngắn:

```text
Khai báo → Lưu Git → Tự kéo → Tự sửa lệch
```

Đây là nền tảng để hiểu các phần sau: **GitHub Actions**, **ArgoCD**, **App of Apps**, **Sync Waves**, và **Rollback**.
