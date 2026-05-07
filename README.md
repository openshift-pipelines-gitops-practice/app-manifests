# app-manifests

OpenShift 上にデプロイするアプリケーション（frontend / backend）の Kubernetes マニフェストを Kustomize で管理するリポジトリです。Argo CD がこのリポジトリを監視し、クラスターへの自動同期（GitOps）を行います。

## ディレクトリ構成

```
app-manifests/
├── base/                        # 共通のベースマニフェスト
│   ├── kustomization.yaml
│   ├── backend/
│   │   ├── deployment.yaml      # Backend Deployment (Spring Boot, port 8080)
│   │   └── service.yaml         # Backend Service
│   └── frontend/
│       ├── deployment.yaml      # Frontend Deployment (Nuxt.js, port 3000)
│       ├── service.yaml         # Frontend Service
│       └── route.yaml           # Frontend Route (TLS edge)
└── envs/                        # 環境別オーバーレイ
    ├── dev/
    │   └── kustomization.yaml   # dev 環境設定
    └── prod/
        └── kustomization.yaml   # prod 環境設定
```

## コンポーネント

| コンポーネント | 説明 | ポート |
|---|---|---|
| backend | Spring Boot アプリケーション（ヘルスチェック: `/actuator/health`） | 8080 |
| frontend | Nuxt.js アプリケーション（環境変数 `NUXT_API_BASE_URL` で backend に接続） | 3000 |

## 環境別設定

### dev 環境 (`envs/dev/`)

- **Namespace**: `dev-handson-app`
- **レプリカ数**: 1（base のまま）
- **イメージレジストリ**: `image-registry.openshift-image-registry.svc:5000/dev-handson-app/`
- **イメージタグ**: CI パイプラインにより commit SHA で自動更新

### prod 環境 (`envs/prod/`)

- **Namespace**: `prod-handson-app`
- **レプリカ数**: 2（JSON Patch でオーバーライド）
- **イメージレジストリ**: `image-registry.openshift-image-registry.svc:5000/prod-handson-app/`
- **リソース増量**:
  - backend: CPU 200m〜1 / Memory 512Mi〜1Gi
  - frontend: CPU 200m〜1 / Memory 256Mi〜512Mi

## イメージタグの更新

CI パイプライン（OpenShift Pipelines）がビルド完了後、環境別の `kustomization.yaml` 内の `newTag` フィールドを commit SHA で更新します。タグの位置は以下のコメントで識別されます:

```yaml
newTag: "<commit-sha>" # backend-image-tag
newTag: "<commit-sha>" # frontend-image-tag
```

## ローカルでのマニフェスト確認

```bash
# dev 環境のマニフェストをレンダリング
kustomize build envs/dev

# prod 環境のマニフェストをレンダリング
kustomize build envs/prod
```
