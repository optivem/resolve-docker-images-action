# Resolve Docker Images Action

[![Test Action](https://github.com/optivem/resolve-docker-images-action/actions/workflows/test.yml/badge.svg)](https://github.com/optivem/resolve-docker-images-action/actions/workflows/test.yml)
[![Release](https://github.com/optivem/resolve-docker-images-action/actions/workflows/release.yml/badge.svg)](https://github.com/optivem/resolve-docker-images-action/actions/workflows/release.yml)

A GitHub Action that resolves base Docker image URLs to versioned URLs and validates that the images exist in the registry.

## Features

- ✅ Resolves base image URLs to versioned URLs with specified tags
- 🔍 Validates that Docker images exist in the registry
- 📦 Returns a JSON array of resolved image URLs
- ⚡ Fast-fail on missing images
- 🐳 Supports multiple Docker registries

## Usage

### Basic Example

```yaml
name: Build and Deploy
on: [push]

jobs:
  resolve-images:
    runs-on: ubuntu-latest
    steps:
      - name: Resolve Docker Images
        id: resolve
        uses: optivem/resolve-docker-images-action@v1
        with:
          tag: 'v1.0.4-rc'
          base-image-urls: |
            myregistry.azurecr.io/myapp/api
            myregistry.azurecr.io/myapp/web
            myregistry.azurecr.io/myapp/worker
      
      - name: Use resolved images
        run: |
          echo "Resolved images: ${{ steps.resolve.outputs.image-urls }}"
```

### Advanced Example with Matrix Strategy

```yaml
name: Multi-Environment Deploy
on: [push]

jobs:
  resolve-images:
    runs-on: ubuntu-latest
    outputs:
      image-urls: ${{ steps.resolve.outputs.image-urls }}
    steps:
      - name: Resolve Docker Images
        id: resolve
        uses: optivem/resolve-docker-images-action@v1
        with:
          tag: ${{ github.sha }}
          base-image-urls: |
            myregistry.azurecr.io/myapp/api
            myregistry.azurecr.io/myapp/web

  deploy:
    needs: resolve-images
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [staging, production]
    steps:
      - name: Deploy to ${{ matrix.environment }}
        run: |
          IMAGES='${{ needs.resolve-images.outputs.image-urls }}'
          echo "Deploying images to ${{ matrix.environment }}: $IMAGES"
```

## Inputs

| Input | Description | Required | Example |
|-------|-------------|----------|---------|
| `tag` | Tag to append to base image URLs | ✅ Yes | `v1.0.4-rc`, `latest`, `${{ github.sha }}` |
| `base-image-urls` | Base image URLs (one per line, without tag) | ✅ Yes | See examples above |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `image-urls` | JSON array of resolved image URLs with tags | `["registry.io/app:v1.0.0","registry.io/web:v1.0.0"]` |

## How It Works

1. **Input Validation**: Validates that both tag and base image URLs are provided
2. **URL Construction**: Appends the specified tag to each base image URL
3. **Image Validation**: Uses `docker manifest inspect` to verify each image exists
4. **Fast Fail**: Stops immediately if any image is not found
5. **Output Generation**: Returns a JSON array of all resolved URLs

## Requirements

- Docker must be available in the runner environment
- Access to the Docker registries containing your images
- Proper authentication for private registries (if applicable)

## Registry Authentication

If you're using private registries, make sure to authenticate before using this action:

```yaml
steps:
  - name: Login to Azure Container Registry
    uses: azure/docker-login@v1
    with:
      login-server: myregistry.azurecr.io
      username: ${{ secrets.ACR_USERNAME }}
      password: ${{ secrets.ACR_PASSWORD }}
  
  - name: Resolve Docker Images
    uses: optivem/resolve-docker-images-action@v1
    with:
      tag: 'v1.0.0'
      base-image-urls: |
        myregistry.azurecr.io/myapp/api
```

## Error Handling

The action will fail with an exit code of 1 if:
- Required inputs are missing or empty
- No valid base image URLs are provided
- Any Docker image cannot be found in the registry
- Docker manifest inspection fails

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
