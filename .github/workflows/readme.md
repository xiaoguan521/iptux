# 使用 GitHub Actions 进行跨架构编译指南

## 基本组件

### 1. 跨平台构建支持
使用 `uraimo/run-on-arch-action@v2` action 来支持在不同CPU架构上进行构建：
- 支持 ARM、ARM64、RISC-V 等多种架构
- 可以在 x86 环境下构建其他架构的软件包

### 2. 基础工作流组件
- 代码检出：`actions/checkout@v4`
- 构建产物上传：`actions/upload-artifact@v3`

## 工作流示例
```yaml
name: build-centos

on:
  workflow_dispatch:
    inputs:
      platform:
        description: '选择构建平台'
        required: true
        default: 'cross-platform'
        type: choice
        options:
          - cross-platform

jobs:
  build-cross-platform:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        include:
          - arch: aarch64
            distro: fedora_latest
    steps:
      
      - uses: uraimo/run-on-arch-action@v2
        name: Build base Fedora aarch64 image
        with:
          arch: ${{ matrix.arch }}
          distro: ${{ matrix.distro }}
          githubToken: ${{ github.token }}
          run: |
            # Fedora 基础系统更新
            dnf -y update
```

## Docker 相关操作

### 1. 使用预构建镜像
```bash
docker pull ghcr.io/xiaoguan521/iptux/run-on-arch-xiaoguan521-iptux-build-centos-aarch64-fedora-latest:latest
```
### 2. 运行跨架构容器
```bash
docker run --name centos-aarch64-fedora-latest -it  --platform linux/arm64   -v $(pwd):/workspace xiaochen1649/ubuntu-aarch64 /bin/bash
``` 

### 3. 设置代理
```bash
export http_proxy=http://host.docker.internal:7890 export https_proxy=http://host.docker.internal:7890
```

## 最佳实践
1. 确保在 workflow 中正确指定目标架构
2. 使用缓存机制加速构建过程
3. 做好错误处理和日志记录
4. 考虑使用矩阵构建同时支持多个架构

## 注意事项
- 确保构建环境中有足够的磁盘空间
- 注意不同架构间的依赖兼容性
- 建议使用容器化构建以确保环境一致性
