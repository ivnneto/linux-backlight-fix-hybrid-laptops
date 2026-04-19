🌐 [English](./README.md) | **Português**

# 🔆 Fix de Brilho no Linux — Notebooks com Gráficos Híbridos (Intel/AMD + NVIDIA)

> Corrige o controle de brilho (teclas Fn + sliders do sistema) no Linux em notebooks com gráficos híbridos.

Problema comum em notebooks gamer e de produtividade da Acer, ASUS, Lenovo, Dell, MSI e outros — especialmente os que combinam GPU integrada Intel/AMD com GPU dedicada NVIDIA.

---

## O problema

Após instalar o Linux, as teclas de brilho (Fn+F5/F6 ou similares) e os sliders de brilho do sistema param de funcionar. A tela fica travada no brilho máximo ou não responde.

**Por que acontece:** Em notebooks com gráficos híbridos, o kernel e o firmware (BIOS/UEFI) frequentemente disputam o controle da interface de backlight. O parâmetro `acpi_backlight` permite definir explicitamente quem assume o controle.

---

## Como corrigir

Edite o arquivo `/etc/default/grub` e localize a linha:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

Adicione o parâmetro adequado para o seu caso (veja as opções abaixo). Exemplo:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=native"
```

Em seguida, regenere a configuração do GRUB:

```bash
# Ubuntu / Pop!_OS / Debian / Mint
sudo update-grub

# Arch / Manjaro / EndeavourOS
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Fedora / RHEL
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

Reinicie o sistema e teste as teclas de brilho.

---

## Qual parâmetro usar?

Tente nessa ordem até um funcionar:

### ✅ `acpi_backlight=native` — comece por aqui
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=native"
```
Instrui o kernel a usar diretamente a interface ACPI nativa do hardware, ignorando o firmware. **Funciona na maioria dos notebooks híbridos modernos** (Intel 10ª geração+, Ryzen 5000+).

---

### `acpi_backlight=vendor`
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=vendor"
```
Usa a implementação ACPI do fabricante do notebook. Tente se `native` não funcionar — comum em modelos mais antigos ou com firmware OEM específico.

---

### `acpi_backlight=video`
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=video"
```
Delega o controle de brilho para o driver da GPU. Pode funcionar bem quando o driver da iGPU Intel/AMD gerencia a saída de vídeo.

---

### `acpi_backlight=none` + `nvidia_drm.modeset=1` — driver proprietário NVIDIA
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=none nvidia_drm.modeset=1"
```
Desativa o backlight ACPI e deixa o driver de modesetting da NVIDIA assumir. Use se estiver com o driver proprietário oficial (não o nouveau) e as opções acima não funcionarem.

---

### `video.use_native_backlight=1` — kernels antigos (< 5.x)
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash video.use_native_backlight=1"
```
Parâmetro mais antigo com efeito similar ao `native`. Útil em kernels anteriores ao 5.x.

---

## Configurações testadas

| Modelo | CPU | GPU | Distro | Kernel | Fix |
|---|---|---|---|---|---|
| Acer Nitro AN515-58 | Intel i5-12500H | RTX 3050 | Ubuntu 22.04, Pop!_OS, Arch | 5.15 – 6.x | `acpi_backlight=native` ✅ |

**Funcionou no seu notebook?** Abre uma PR ou issue e adiciono na tabela!

---

## Ainda não funcionou?

- **Secure Boot ativado?** Alguns sistemas ignoram parâmetros do kernel com Secure Boot ligado. Desative na BIOS e tente novamente.
- **Usando Wayland?** Tente mudar para uma sessão X11 primeiro para descartar problemas do compositor.
- **Driver proprietário da NVIDIA?** Não misture `acpi_backlight=native` com `nvidia_drm.modeset=1` — use a combinação dedicada acima.
- **Verifique o que o sistema reporta:**
  ```bash
  ls /sys/class/backlight/
  cat /sys/class/backlight/*/brightness
  ```
  Se o diretório estiver vazio, tente `acpi_backlight=vendor` ou `none`.

---

## Contribuindo

Se funcionou em um notebook não listado acima, abre uma issue ou PR com:

- Modelo do notebook
- CPU e GPU
- Distro e versão do kernel (`uname -r`)
- O parâmetro que funcionou

---

## Referências

- [kernel.org — parâmetros do kernel](https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html)
- [ArchWiki — Backlight](https://wiki.archlinux.org/title/Backlight)
- [ArchWiki — NVIDIA DRM kernel mode setting](https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting)

