# contanki (fork com patch)

Fork local do [Contanki](https://github.com/roxgib/anki-contanki) v1.0.2 (suporte a controles/gamepads no Anki),
que está sem releases desde fevereiro de 2024. Licença: **AGPL-3.0** (veja `LICENSE`), como o original.
Autor original: Sam Bradshaw.

## Instalar (Linux)

```sh
git clone git@github.com:costaluu/contanki.git && cd contanki
./install.py --dry-run     # mostra o que será feito
./install.py               # instala no Flatpak e/ou em ~/.local/share/Anki2
```

Feche o Anki antes e abra de novo depois. Opções: `--list`, `--target flatpak|native`, `--dest PASTA_addons21`,
`--link` (symlink, para desenvolver), `--uninstall`, `--no-flatpak-override`. Só usa a biblioteca padrão do Python 3.

O instalador:
- move a versão anterior para `~/.local/share/anki-addons-backups/<data>/` (nunca deixa backup dentro de `addons21`,
  o Anki trata toda pasta ali como extensão);
- mantém o `meta.json` (configuração) e o `user_files/`;
- marca `"update_enabled": false` para o AnkiWeb não sobrescrever o patch;
- no Flatpak, roda `flatpak override --user --filesystem=/run/udev:ro net.ankiweb.Anki`. Sem isso a sandbox não vê o
  banco do udev e o Chromium não reconhece o controle. Desfazer: `flatpak override --user --reset net.ankiweb.Anki`.

A pasta instalada é `1898790263` (o id do AnkiWeb) de propósito, para não coexistir com o original.

## Mudança em relação ao original

`addon/controller.js` (`patches/contanki-rescan.patch`): o original detecta o controle só pelo evento
`gamepadconnected`, que dispara uma vez. Se a página recarrega ou o listener entra depois do evento, o controle nunca
conecta. O patch adiciona `rescan()`: a cada 1 s, enquanto não há controle conectado, chama `getGamepads()` e conecta
se achar um (nova tentativa a cada 5 s).

## Notas sobre controle Bluetooth (Xbox)
- Em container rootless (Distrobox/Toolbox) a `libudev` rejeita os eventos de hotplug, então um controle religado com o
  Anki aberto não é visto. No Flatpak, com `/run/udev` exposto, funciona (testado: 42 eventos aceitos).
- O Chromium só expõe o controle depois do primeiro botão apertado com a janela do Anki em foco.
- Testado com Anki 26.09.3 (Flatpak) e Xbox Wireless Controller (045e:02fd).
