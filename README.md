<p align="center">
  <img src="res/logo-header.svg" alt="RustDesk - Your remote desktop"><br>
  <a href="#public-servers">Servers</a> •
  <a href="#raw-steps-to-build">Build</a> •
  <a href="#how-to-build-with-docker">Docker</a> •
  <a href="#file-structure">Structure</a> •
  <a href="#snapshot">Snapshot</a><br>
</p>

Mais um software de desktop remoto, escrito em Rust. Funciona por padrão, sem necessidade de configuração. Você tem completo controle de seus dados, sem se preocupar com segurança. 

![image](https://user-images.githubusercontent.com/71636191/171661982-430285f0-2e12-4b1d-9957-4a58e375304d.png)

## Dependências

Versões de desktop utilizam [sciter](https://sciter.com/) para a GUI, por favor baixe a biblioteca dinâmica sciter por conta própria.

[Windows](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.win/x64/sciter.dll) |
[Linux](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.lnx/x64/libsciter-gtk.so) |
[MacOS](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.osx/libsciter.dylib)

## Compilação crua

- Prepare seu ambiente de desenvolvimento Rust e ambiente de compilação C++

- Instale [vcpkg](https://github.com/microsoft/vcpkg), e configure a variável de ambiente `VCPKG_ROOT` corretamente

  - Windows: vcpkg install libvpx:x64-windows-static libyuv:x64-windows-static opus:x64-windows-static aom:x64-windows-static
  - Linux/MacOS: vcpkg install libvpx libyuv opus aom

- Execute `cargo run`

## Como compilar no Linux

### Ubuntu 18 (Debian 10)

```sh
sudo apt install -y g++ gcc git curl wget nasm yasm libgtk-3-dev clang libxcb-randr0-dev libxdo-dev libxfixes-dev libxcb-shape0-dev libxcb-xfixes0-dev libasound2-dev libpulse-dev cmake
```

### Fedora 28 (CentOS 8)

```sh
sudo yum -y install gcc-c++ git curl wget nasm yasm gcc gtk3-devel clang libxcb-devel libxdo-devel libXfixes-devel pulseaudio-libs-devel cmake alsa-lib-devel
```

### Arch (Manjaro)

```sh
sudo pacman -Syu --needed unzip git cmake gcc curl wget yasm nasm zip make pkg-config clang gtk3 xdotool libxcb libxfixes alsa-lib pipewire
```

### Instale vcpkg

```sh
git clone https://github.com/microsoft/vcpkg
cd vcpkg
git checkout 2023.04.15
cd ..
vcpkg/bootstrap-vcpkg.sh
export VCPKG_ROOT=$HOME/vcpkg
vcpkg/vcpkg install libvpx libyuv opus aom
```

### Conserte libvpx (Para o Fedora)

```sh
cd vcpkg/buildtrees/libvpx/src
cd *
./configure
sed -i 's/CFLAGS+=-I/CFLAGS+=-fPIC -I/g' Makefile
sed -i 's/CXXFLAGS+=-I/CXXFLAGS+=-fPIC -I/g' Makefile
make
cp libvpx.a $HOME/vcpkg/installed/x64-linux/lib/
cd
```

### Compile

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
git clone https://github.com/rustdesk/rustdesk
cd rustdesk
mkdir -p target/debug
wget https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.lnx/x64/libsciter-gtk.so
mv libsciter-gtk.so target/debug
VCPKG_ROOT=$HOME/vcpkg cargo run
```

## Como compilar com Docker

Comece clonando o repositório e montando o container docker:

```sh
git clone https://github.com/rustdesk/rustdesk
cd rustdesk
docker build -t "rustdesk-builder" .
```

Então, sempre que precisar compilar a aplicação, execute este comando:

```sh
docker run --rm -it -v $PWD:/home/user/rustdesk -v rustdesk-git-cache:/home/user/.cargo/git -v rustdesk-registry-cache:/home/user/.cargo/registry -e PUID="$(id -u)" -e PGID="$(id -g)" rustdesk-builder
```

Note que a primeira compilação pode demorar mais antes que as dependências sejam armazenadas em cache, as compilações subsequentes serão mais rápidas. Adicionalmente, se você precisar especificar argumentos diferentes para o comando de compilação, você pode fazê-lo ao final do comando na posição do `<OPTIONAL-ARGS>`. Por exemplo, se você gostaria de compilar uma versão de release otimizada, você executaria o comando acima seguido de `--release`. O executável gerado estará disponível no diretório alvo no seu sistema, e pode ser executado com:

```sh
target/debug/rustdesk
```

Ou, se estiver rodando um executável de release:

```sh
target/release/rustdesk
```

Por favor verifique que está executando estes comandos da raiz do repositório do RustDesk, senão a aplicação pode não encontrar os recursos necessários. Note também que outros subcomandos do cargo como `install` ou `run` não são suportados atualmente via este método, já que eles iriam instalar ou rodar o programa dentro do container ao invés do host.

## Estrutura de arquivos

- **[libs/hbb_common](https://github.com/rustdesk/rustdesk/tree/master/libs/hbb_common)**: codec de vídeo, configurações, wrapper de tcp/udp, protobuf, funções de sistema de arquivos para transferência de arquivos, e outras funções utilitárias
- **[libs/scrap](https://github.com/rustdesk/rustdesk/tree/master/libs/scrap)**: captura de tela
- **[libs/enigo](https://github.com/rustdesk/rustdesk/tree/master/libs/enigo)**: controle de teclado/mouse específico a cada plataforma
- **[src/ui](https://github.com/rustdesk/rustdesk/tree/master/src/ui)**: GUI
- **[src/server](https://github.com/rustdesk/rustdesk/tree/master/src/server)**: serviços de áudio/área de transferência/entrada/vídeo, e conexões de rede
- **[src/client.rs](https://github.com/rustdesk/rustdesk/tree/master/src/client.rs)**: iniciar uma conexão "peer to peer"
- **[src/rendezvous_mediator.rs](https://github.com/rustdesk/rustdesk/tree/master/src/rendezvous_mediator.rs)**: Comunicação com [rustdesk-server](https://github.com/rustdesk/rustdesk-server), aguardar pela conexão remota direta (TCP hole punching) ou conexão indireta (relayed)
- **[src/platform](https://github.com/rustdesk/rustdesk/tree/master/src/platform)**: código específico a cada plataforma

## Screenshots

![image](https://user-images.githubusercontent.com/71636191/113112362-ae4deb80-923b-11eb-957d-ff88daad4f06.png)

![image](https://user-images.githubusercontent.com/71636191/113112619-f705a480-923b-11eb-911d-97e984ef52b6.png)

![image](https://user-images.githubusercontent.com/71636191/113112857-3fbd5d80-923c-11eb-9836-768325faf906.png)

![image](https://user-images.githubusercontent.com/71636191/135385039-38fdbd72-379a-422d-b97f-33df71fb1cec.png)
