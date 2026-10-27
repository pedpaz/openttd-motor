# openttd-motor

Nosso motor do jogo: uma cópia do [JGR Patchpack](https://github.com/JGRennison/OpenTTD-patches) (OpenTTD com frenagem realista, quadro de horários, restrições de rota e muito mais), onde entram as mudanças de realismo e de gráficos.

- **Versão para Windows:** a cada envio, o GitHub compila e publica em *Releases → versao-teste* um `.zip` pronto (extrair e abrir `openttd.exe`). Já vem com o OpenGFX e o mod de veículos [openttd-frota](https://github.com/pedpaz/openttd-frota) ligado.
- **Atualizar com o JGR:** `git fetch https://github.com/JGRennison/OpenTTD-patches jgrpp && git merge FETCH_HEAD`.
- **Regras de realismo** (carga perecível, combustível, oficina): ver [docs/pedro-realismo.md](docs/pedro-realismo.md). Tudo em Configurações → Veículos, ligado por padrão.
