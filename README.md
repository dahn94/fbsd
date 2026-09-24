# fbsd

VMs FreeBSD arm64 no macOS (Apple Silicon) com QEMU + HVF, num único script de shell.

Pensado para desenvolvimento: a VM roda sem interface gráfica, você entra por SSH e edita o código no Mac numa pasta compartilhada com a VM.

```sh
fbsd create dev --share ~/code
fbsd start dev
fbsd ssh dev
```

## O que ele faz

- Baixa a **imagem oficial** do FreeBSD (`BASIC-CLOUDINIT`, qcow2) de download.freebsd.org, confere o SHA256 e guarda em cache.
- Configura o primeiro boot com o **nuageinit** (o cloud-init nativo do FreeBSD): cria seu usuário com a sua chave SSH, sudo sem senha, e instala `git` e `sudo`.
- Roda com **aceleração nativa** (`-accel hvf -cpu host`), então fica perto da velocidade nativa.
- **Pasta compartilhada** via virtio-9p (`p9fs`, disponível a partir do FreeBSD 15.0): a pasta do Mac aparece em `~/src` na VM, com o mesmo UID, para não haver briga de permissões.
- Aplica `kern.hz=100` para a VM não consumir CPU parada.
- Rede via NAT do QEMU, com o SSH exposto só em `127.0.0.1`.

## Requisitos

- Mac com Apple Silicon (M1 ou posterior)
- [Homebrew](https://brew.sh)
- Uma chave SSH (`ssh-keygen -t ed25519`, se ainda não tiver)

```sh
brew install qemu xz xorriso
```

## Instalação

```sh
mkdir -p ~/bin
curl -fsSL https://raw.githubusercontent.com/SEU_USUARIO/fbsd/main/fbsd -o ~/bin/fbsd
chmod +x ~/bin/fbsd
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

## Uso

| Comando | O que faz |
|---|---|
| `fbsd create NOME [opções]` | Cria a VM (baixa a imagem se precisar) |
| `fbsd start NOME [--fg]` | Sobe em background e espera o SSH; `--fg` abre o console no terminal (sair: Ctrl-A, X) |
| `fbsd stop NOME [--force]` | Faz um shutdown limpo; `--force` mata o QEMU |
| `fbsd ssh NOME [cmd...]` | Abre SSH na VM ou roda um comando nela |
| `fbsd console NOME` | Acompanha o log do console serial |
| `fbsd ssh-config NOME` | Imprime um bloco para colar no `~/.ssh/config` |
| `fbsd list` | Lista as VMs e o estado de cada uma |
| `fbsd destroy NOME` | Apaga a VM (pede confirmação) |

### Opções do `create`

| Opção | Padrão |
|---|---|
| `--version X.Y` | `15.1` |
| `--fs zfs\|ufs` | `zfs` |
| `--disk TAM` | `40G` |
| `--mem TAM` | `8G` |
| `--cpus N` | núcleos de performance do Mac |
| `--share DIR` | nenhuma |
| `--port N` | primeira porta livre a partir de 2222 |
| `--user NOME` | seu usuário do macOS |

### VS Code / SSH direto

```sh
fbsd ssh-config dev >> ~/.ssh/config
ssh dev
```

Depois disso, a VM aparece no Remote-SSH do VS Code como `dev`.

## Onde ficam os arquivos

```
~/.fbsd-vm/
├── cache/            imagens base descompactadas (reutilizadas entre VMs)
└── dev/
    ├── disk.qcow2    disco da VM
    ├── seed.iso      configuração de primeiro boot (cidata)
    ├── vm.conf       CPUs, memória, porta, pasta compartilhada
    └── console.log   saída do console serial
```

O diretório base muda com `FBSD_HOME`, e o mirror com `FBSD_MIRROR`.

## Atualizando o FreeBSD na VM

```sh
sudo freebsd-update fetch install
sudo pkg upgrade
```

## Problemas comuns

- **O SSH não responde no `start`:** o primeiro boot instala pacotes e pode levar alguns minutos. Acompanhe com `fbsd console NOME`.
- **A pasta compartilhada não aparece:** o `p9fs` exige FreeBSD 15.0 ou posterior. Nas versões 14.x, use NFS ou `rsync`.
- **A VM consome CPU parada:** confira se `kern.hz=100` está em `/boot/loader.conf`.

## Licença

BSD 2-Clause. Veja [LICENSE](LICENSE).
