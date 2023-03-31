# monitor_virtual_os_15.4_nv_prime
Habilitando monitor virtual em notebook com placa de vídeo nVidia (Prime) em openSUSE Leap 15.4

<h4>Configuração do sistema</h4>

![image](https://user-images.githubusercontent.com/39818426/228973314-c1f4cd0b-3bd0-4a20-950c-4f7a25b81377.png)

<h4>Versão do driver da nVidia</h4>

![image](https://user-images.githubusercontent.com/39818426/228974570-d683bd3c-06b3-4f79-bb17-840d09f4fb57.png)


<p>A instalação do driver da nvidia foi feita pelo repositório padrão da nvidia.</p>
https://download.nvidia.com/opensuse/leap/15.4</br></br>

<p>Logo após a instalação do driver oficial, veio o primeiro problema: Não reconhecia a tela do notebook, somente o monitor externo. Para resolver, foi preciso acrescentar a seguintes linhas ao final do arquivo /etc/X11/xorg.conf:</p>
<pre><code>Section "Device"
    Identifier  "intel"
    Driver      "modesetting"
EndSection
Section "Screen"
    Identifier "intel"
    Device "intel"
EndSection</code></pre>
<p>Após inserir as linha, fazer logout e login que a configuração já será aplicada.</p>
<p>Agora, listar as saídas com o comando <code>xrandr --listproviders</code>:</p>
<pre><code>Providers: number : 2
Provider 0: id: 0x247; cap: 0x1 (Source Output); crtcs: 4; outputs: 3; associated providers: 1; name: NVIDIA-0
    output HDMI-0
    output DP-0
    output DP-1
Provider 1: id: 0x47; cap: 0xb (Source Output, Sink Output, Sink Offload); crtcs: 6; outputs: 1; associated providers: 1; name: Intel
    output eDP1</code></pre>
<p>Como pode ser visto, não há monitores virutais. Para habilitar, precisa criar um arquivo para que o Xorg configure os monitores virtuais na pasta /etc/X11/xorg.conf.d/.</p>
/etc/X11/xorg.conf.d/30-virtscreen.conf
<pre><code>Section "Device"
  Identifier "nvidiagpu0"
  Driver     "nvidia" # Because you are using Nvidia proprietary driver. Change to "nouveau" if you are using open source nouveau driver
EndSection
# Then configure intel internal GPU
Section "Device"
  Identifier "intelgpu0"
  Driver     "intel"
  Option     "VirtualHeads" "2"
EndSection
</code></pre>
<p>Novamente, após inserir essas linha, só fazer logout e login que a configuração já será aplicada.</p>
<p>Listar novamente as saídas com o comando <code>xrandr --listproviders</code> para ver se monitores virtuais estão habilitados:</p>
<pre><code>Providers: number : 2
Provider 0: id: 0x247; cap: 0x1 (Source Output); crtcs: 4; outputs: 3; associated providers: 1; name: NVIDIA-0
    output HDMI-0
    output DP-0
    output DP-1
Provider 1: id: 0x47; cap: 0xb (Source Output, Sink Output, Sink Offload); crtcs: 6; outputs: 1; associated providers: 1; name: Intel
    output eDP1</code></pre>
<p>Nesse caso, não apareceu. Após verificar os logs do Xorg, o módulo 'intel' constava com inexistente:</p>
<code>sudo cat /var/log/Xorg.0.log</code>
<pre><code>[   754.218] (II) LoadModule: "intel"
[   754.219] (WW) Warning, couldn't open module intel
[   754.219] (EE) Failed to load module "intel" (module does not exist, 0)
</code></pre>

<p>Para resolver, foi preciso instalar o parcote <code>xf86-video-intel</code>. Após a instalação, mais uma vez, logout e login e a configuração foi aplicada.</p>
<p>Verificar os logs do Xorg, o módulo 'intel' agora foi carregado.</p>
<pre><code>[  3249.060] (II) LoadModule: "intel"
[  3249.061] (II) Loading /usr/lib64/xorg/modules/drivers/intel_drv.so
[  3249.061] (II) Module intel: vendor="X.Org Foundation"
</code></pre>

<p>Listando novamente as saídas com o comando <code>xrandr --listproviders</code>, agora os monitores virtuais estão habilitados:</p>
<pre><code>Providers: number : 2
Provider 0: id: 0x247; cap: 0x1 (Source Output); crtcs: 4; outputs: 3; associated providers: 1; name: NVIDIA-0
    output HDMI-0
    output DP-0
    output DP-1
Provider 1: id: 0x47; cap: 0xb (Source Output, Sink Output, Sink Offload); crtcs: 6; outputs: 4; associated providers: 1; name: Intel
    output eDP1
    output VIRTUAL1
    output VIRTUAL2
    output VIRTUAL3</code></pre>

<h4>Alguns comandos para usar os monitores virtuais:</h4>
<pre><code>gtf 1920 1080 60
xrandr --newmode "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
xrandr --addmode VIRTUAL1 1920x1080_60.00
xrandr --output VIRTUAL1 --off
</code></pre>
