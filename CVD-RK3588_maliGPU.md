# 3588 GPU加速

## 要求

1. RK 3588 系列SOC
2. armbian 6.18+内核
3. 编译mesa驱动

## 编译mesa驱动

```bash
sudo apt-get install -y \
    build-essential \
    git \
    ninja-build \
    pkg-config \
    bison \
    flex

# 安装 Mesa 编译依赖
sudo apt-get install -y \
    libdrm-dev \
    libx11-dev \
    libxext-dev \
    libxxf86vm-dev \
    libxcb-dri2-0-dev \
    libxcb-dri3-dev \
    libxcb-present-dev \
    libxshmfence-dev \
    libwayland-dev \
    wayland-protocols \
    libwayland-egl-backend-dev \
    libx11-xcb-dev \
    libxcb-keysyms1-dev \
    libxrandr-dev \
    libexpat1-dev \
    libclang-18-dev \
    libllvmspirvlib-18-dev \
    llvm-18 \
    llvm-18-dev \
    libclc-18-dev \
    glslang-dev

sudo apt-get install -y libxcb-cursor0 \
    libxcb-icccm4 \
    libxcb-image0 \
    libxcb-keysyms1 \
    libxcb-randr0 \
    libxcb-render-util0 \
    libxcb-render0 \
    libxcb-shape0 \
    libxcb-sync1 \
    libxcb-util1 \
    libxcb-xfixes0 \
    libxcb-xinerama0 \
    libxcb-xkb1 \
    libxkbcommon-x11-0

mkdir ${HOME}/mesa ; cd ${HOME}/mesa
uv venv
source .venv/bin/activate
uv pip install meson mako pyyaml packaging
meson --version  # 需要 >= 1.4.0

git clone https://gitlab.freedesktop.org/mesa/mesa.git mesa-src

meson setup build \
    --prefix=${HOME}/mesa/mesa-install \
    --libdir=lib \
    -Dgallium-drivers=panfrost \
    -Dvulkan-drivers=panfrost \
    -Dplatforms=x11 \
    -Dbuildtype=release \
    -Dgles1=disabled \
    -Dgles2=enabled \
    -Dglx=dri \
    -Dllvm=enabled \
    -Dshared-llvm=enabled \
    -Degl=enabled \
    -Dgbm=enabled

ninja -C build
ninja -C build install

cd /your/cvd/home
# 注意这个lc命令中，HOME目录是需要指定绝对值的，因为权限切换了，HOME路径变成/root了
sudo bash -lc '
ulimit -n 1048576
export HOME=/your/cvd/home
export MESA_PREFIX=/you/mesa/mesa-install
export LD_LIBRARY_PATH="${MESA_PREFIX}/lib:${LD_LIBRARY_PATH}"
export LIBGL_DRIVERS_PATH="${MESA_PREFIX}/lib/dri"
export EGL_DRIVERS_PATH="${MESA_PREFIX}/lib"
export VK_ICD_FILENAMES="${MESA_PREFIX}/share/vulkan/icd.d/panfrost_icd.aarch64.json"
export GBM_DRIVERS_PATH="${MESA_PREFIX}/lib/gbm"
export EGL_PLATFORM=surfaceless
export DISPLAY=:0
bin/launch_cvd --gpu_mode=gfxstream
'
```
<img width="794" height="989" alt="c64300ac8531ceabb9162f6d28b54f91" src="https://github.com/user-attachments/assets/df8ce678-8924-4432-afc5-ed9df383ac91" />

