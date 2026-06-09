Nội dung tiếp theo sau **GitOps Principles** là **GitHub Actions & CI/CD**.

Hiểu đơn giản: **GitHub Actions là công cụ tự động hóa nằm trong GitHub**. Nó giúp mình tự động chạy các việc như:

```text
Push code lên GitHub
→ tự build
→ tự test
→ tự kiểm tra YAML/Kubernetes manifest
→ tự tạo plan
→ merge vào main thì tự deploy/apply
```

GitHub Actions được mô tả là nền tảng **CI/CD** cho phép tự động hóa build, test và deployment pipeline. Nó có thể chạy workflow khi có pull request hoặc khi pull request đã merge. 

---

# 1. CI/CD là gì?

Trước khi hiểu GitHub Actions, phải hiểu **CI/CD**.

## CI = Continuous Integration

**CI** nghĩa là “tích hợp liên tục”.

Hiểu đơn giản:

```text
Mỗi lần bạn đẩy code lên GitHub
→ hệ thống tự kiểm tra code có lỗi không
→ tự chạy test
→ tự báo pass/fail
```

Ví dụ bạn tạo pull request:

```text
feature branch → pull request vào main
```

GitHub Actions có thể tự chạy:

```bash
npm test
docker build
kubectl diff
terraform plan
```

Nếu lỗi thì PR bị đỏ, chưa nên merge.

**Câu dễ nhớ:**

```text
CI = kiểm tra trước khi nhập code vào nhánh chính.
```

---

## CD = Continuous Delivery / Continuous Deployment

**CD** là bước sau CI.

Có 2 cách hiểu:

```text
Continuous Delivery = chuẩn bị sẵn để deploy, nhưng có thể cần người bấm nút.
Continuous Deployment = merge xong là tự deploy luôn.
```

Trong W9 D1, phần quan trọng là:

```text
plan-on-PR + apply-on-merge
```

Nghĩa là:

```text
Pull Request → chỉ plan/check, chưa deploy thật
Merge vào main → mới apply/deploy thật
```

---

# 2. GitHub Actions dùng để làm gì trong W9 D1?

Trong bài W9, GitHub Actions không phải là tool deploy chính vào cluster theo kiểu truyền thống. Nó chủ yếu dùng để:

```text
1. Kiểm tra code/config khi có Pull Request
2. Chạy plan để xem thay đổi sẽ làm gì
3. Chỉ cho merge khi kiểm tra pass
4. Sau khi merge, cập nhật trạng thái trong Git
5. ArgoCD sẽ tự kéo thay đổi từ Git về cluster
```

Tức là GitHub Actions đứng ở phía **CI/CD pipeline**, còn ArgoCD đứng ở phía **GitOps sync**.

Luồng đúng nên hiểu là:

```text
Developer sửa YAML
→ tạo Pull Request
→ GitHub Actions chạy check/plan
→ reviewer xem kết quả
→ merge vào main
→ ArgoCD phát hiện Git thay đổi
→ ArgoCD sync vào Kubernetes
```

---

# 3. Các thành phần chính của GitHub Actions

GitHub Actions có 5 khái niệm phải nhớ:

```text
Workflow
Event
Job
Step
Runner
```

Nguồn GitHub Actions giải thích rằng workflow được trigger bởi event; workflow chứa một hoặc nhiều job; mỗi job chạy trên runner; mỗi job có nhiều step; step có thể chạy script hoặc action tái sử dụng. 

---

# 4. Workflow là gì?

**Workflow = quy trình tự động.**

Ví dụ:

```text
Khi có Pull Request
→ checkout code
→ kiểm tra YAML
→ chạy test
→ chạy terraform plan hoặc kubectl diff
```

Workflow được viết bằng file YAML trong thư mục:

```text
.github/workflows/
```

Ví dụ:

```text
.github/workflows/pr-check.yml
.github/workflows/deploy.yml
```

GitHub Docs nói workflow là một process tự động, được định nghĩa bằng file YAML trong repository, có thể chạy khi có event, chạy thủ công hoặc chạy theo lịch. 

Ví dụ workflow đơn giản:

```yaml
name: PR Check

on:
  pull_request:
    branches:
      - main

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Validate YAML
        run: |
          echo "Check Kubernetes manifests"
```

---

# 5. Event là gì?

**Event = sự kiện kích hoạt workflow.**

Ví dụ các event thường gặp:

```text
push
pull_request
workflow_dispatch
schedule
release
```

Trong W9 D1, quan trọng nhất là:

```text
pull_request
push
workflow_dispatch
```

## pull_request

Chạy khi có pull request.

Dùng cho:

```text
plan-on-PR
```

Ví dụ:

```yaml
on:
  pull_request:
    branches:
      - main
```

Nghĩa là khi ai đó mở PR vào `main`, workflow sẽ chạy.

GitHub Docs nói `pull_request` thường chạy khi pull request được mở, mở lại hoặc branch của PR được cập nhật. 

---

## push

Chạy khi có commit được push.

Trong W9, thường dùng cho nhánh `main`:

```yaml
on:
  push:
    branches:
      - main
```

Nghĩa là khi code đã merge vào `main`, workflow chạy.

GitHub Docs nói `push` chạy khi bạn push commit hoặc tag vào repository. 

---

## workflow_dispatch

Chạy thủ công bằng nút **Run workflow** trên GitHub.

Ví dụ:

```yaml
on:
  workflow_dispatch:
```

Dùng khi mình muốn tự bấm chạy lại pipeline.

GitHub Docs nói `workflow_dispatch` cho phép trigger workflow thủ công bằng GitHub API, GitHub CLI hoặc GitHub UI. 

---

# 6. Job là gì?

**Job = một nhóm việc chạy trên cùng một máy runner.**

Ví dụ workflow có 2 job:

```text
Job 1: lint
Job 2: test
```

Mặc định các job có thể chạy song song. Nếu muốn job sau chờ job trước, dùng `needs`.

Ví dụ:

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Lint YAML"

  plan:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - run: echo "Run plan"
```

Ở đây:

```text
plan phải chờ lint chạy xong.
```

GitHub Docs nói job là tập hợp các step trong workflow, chạy trên cùng runner; các job mặc định không phụ thuộc nhau và chạy song song, nhưng có thể cấu hình dependency. 

---

# 7. Step là gì?

**Step = từng bước nhỏ trong job.**

Ví dụ:

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Run test
    run: npm test
```

Có 2 kiểu step:

```text
uses = dùng action có sẵn
run  = chạy lệnh shell
```

Ví dụ `uses`:

```yaml
- uses: actions/checkout@v4
```

Nghĩa là dùng action có sẵn để tải code repo về runner.

Ví dụ `run`:

```yaml
- run: kubectl apply --dry-run=client -f k8s/
```

Nghĩa là chạy lệnh trực tiếp.

GitHub Docs nói step có thể là shell script hoặc một action có thể tái sử dụng. 

---

# 8. Runner là gì?

**Runner = máy chạy workflow.**

Hiểu đơn giản:

```text
GitHub Actions cần một cái máy để chạy lệnh.
Cái máy đó gọi là runner.
```

Có 2 loại:

```text
GitHub-hosted runner = máy do GitHub cấp
Self-hosted runner = máy mình tự cài
```

Ví dụ:

```yaml
runs-on: ubuntu-latest
```

Nghĩa là job này chạy trên máy Ubuntu do GitHub cấp.

GitHub Docs nói runner là server chạy workflow khi workflow được trigger; mỗi runner chạy một job tại một thời điểm, và GitHub cung cấp runner Ubuntu, Windows, macOS. 

---

# 9. Action là gì?

**Action = khối lệnh/tác vụ có sẵn để tái sử dụng.**

Ví dụ thay vì tự viết lệnh tải code, dùng:

```yaml
- uses: actions/checkout@v4
```

Ví dụ setup Node.js:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
```

GitHub Docs nói action là tập hợp job/code tái sử dụng, giúp giảm việc viết lặp lại trong workflow. 

---

# 10. Cấu trúc YAML cơ bản của GitHub Actions

Một workflow thường có dạng:

```yaml
name: D1 GitOps CI

on:
  pull_request:
    branches:
      - main

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Validate manifests
        run: |
          echo "Validate Kubernetes YAML"
```

Giải thích từng dòng:

```text
name
→ tên workflow hiển thị trên tab Actions

on
→ workflow chạy khi nào

jobs
→ workflow gồm những job nào

runs-on
→ job chạy trên máy nào

steps
→ các bước trong job

uses
→ dùng action có sẵn

run
→ chạy lệnh shell
```

Workflow file phải nằm trong:

```text
.github/workflows/
```

GitHub Docs nói workflow file dùng YAML, có đuôi `.yml` hoặc `.yaml`, và phải đặt trong `.github/workflows`. 

---

# 11. plan-on-PR là gì?

**plan-on-PR = khi mở Pull Request thì chỉ kiểm tra/kế hoạch, chưa apply thật.**

Ví dụ bạn sửa file Kubernetes:

```yaml
replicas: 3
```

Thay vì deploy ngay, GitHub Actions sẽ kiểm tra:

```text
YAML có hợp lệ không?
Manifest có lỗi không?
Kustomize/Helm render được không?
Terraform plan có thay đổi gì?
```

Ví dụ workflow:

```yaml
name: Plan on PR

on:
  pull_request:
    branches:
      - main

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Show planned changes
        run: |
          echo "Run validation / diff / plan here"
```

Mục tiêu:

```text
PR chỉ được merge nếu plan/check pass.
```

**Câu dễ nhớ:**

```text
plan-on-PR = mở PR thì xem trước thay đổi, chưa đụng thật vào production/cluster.
```

---

# 12. apply-on-merge là gì?

**apply-on-merge = merge vào main rồi mới áp dụng thay đổi.**

Ví dụ:

```yaml
on:
  push:
    branches:
      - main
```

Khi PR đã merge vào `main`, workflow chạy.

Trong GitOps, workflow này có thể:

```text
build image
push image
update manifest trong Git
hoặc notify ArgoCD
```

Nhưng điểm quan trọng là:

```text
Không deploy tùy tiện từ branch lẻ.
Chỉ main mới là nguồn chuẩn.
```

Ví dụ đơn giản:

```yaml
name: Apply on Merge

on:
  push:
    branches:
      - main

jobs:
  apply:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Apply after merge
        run: |
          echo "This runs only after merge to main"
```

**Câu dễ nhớ:**

```text
apply-on-merge = merge vào main xong mới cho thay đổi có hiệu lực.
```

---

# 13. Vì sao không apply ngay khi Pull Request?

Vì PR chưa chắc đúng.

Nếu apply ngay khi PR:

```text
Người A mở PR lỗi
→ cluster bị deploy bản lỗi
→ app chết
```

Cách đúng:

```text
PR chỉ check/plan
Reviewer xem
Test pass
Merge vào main
Sau đó mới deploy/sync
```

Luồng an toàn:

```text
Pull Request = nháp, kiểm tra
main = bản chính, được phép chạy thật
```

---

# 14. GitHub Actions liên quan gì đến GitOps?

GitHub Actions và GitOps không thay thế nhau. Chúng phối hợp với nhau.

```text
GitHub Actions = kiểm tra, build, test, plan
ArgoCD/Flux = sync Git vào Kubernetes
Git = source of truth
```

Luồng chuẩn trong W9:

```text
1. Developer sửa manifest
2. Push branch
3. Mở Pull Request
4. GitHub Actions chạy plan/check
5. Merge vào main
6. ArgoCD thấy Git thay đổi
7. ArgoCD sync cluster
8. Nếu cluster bị sửa tay, ArgoCD kéo lại đúng Git
```

---

# 15. Ví dụ thực tế cho W9 day-a

Cấu trúc repo:

```text
cloud/
  w9/
    day-a/
      .github/
        workflows/
          plan-on-pr.yml
          apply-on-merge.yml
      argocd/
        application.yaml
      k8s/
        deployment.yaml
        service.yaml
```

File `plan-on-pr.yml`:

```yaml
name: Plan on PR

on:
  pull_request:
    branches:
      - main
    paths:
      - "cloud/w9/day-a/**"

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Validate YAML files
        run: |
          echo "Validate YAML/K8s manifests"
```

File `apply-on-merge.yml`:

```yaml
name: Apply on Merge

on:
  push:
    branches:
      - main
    paths:
      - "cloud/w9/day-a/**"

jobs:
  apply:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Apply after merge
        run: |
          echo "This runs after merge to main"
```

Trong GitHub Actions, `paths` giúp workflow chỉ chạy khi file liên quan thay đổi. GitHub Docs nói có thể dùng path filter với `push` và `pull_request` để workflow chỉ chạy khi các file nhất định thay đổi. 

---

# 16. Những từ cần thuộc

```text
Workflow
= quy trình tự động trong GitHub Actions

Event
= sự kiện kích hoạt workflow

Job
= nhóm step chạy trên cùng runner

Step
= từng bước nhỏ trong job

Runner
= máy chạy job

Action
= tác vụ có sẵn để tái sử dụng

CI
= kiểm tra code tự động

CD
= triển khai/phát hành tự động

plan-on-PR
= PR chỉ kiểm tra/xem trước, không apply thật

apply-on-merge
= merge vào main rồi mới apply/deploy
```

---

# 17. Chốt lại phần GitHub Actions

Bạn chỉ cần nhớ đoạn này:

```text
GitHub Actions là công cụ CI/CD của GitHub.
Nó chạy workflow tự động khi có event như pull_request hoặc push.
Workflow gồm nhiều job.
Job gồm nhiều step.
Job chạy trên runner.
Step có thể dùng action có sẵn hoặc chạy lệnh shell.
```

Trong W9 D1:

```text
Pull Request → GitHub Actions chạy plan/check
Merge vào main → thay đổi mới được apply/sync
ArgoCD/Flux → tự kéo desired state từ Git về cluster
```

Cách nhớ siêu ngắn:

```text
GitHub Actions = người kiểm tra cổng Git.
ArgoCD = người đồng bộ cluster theo Git.
Git = bản thiết kế chuẩn.
```
