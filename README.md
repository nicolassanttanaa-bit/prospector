Aqui está o passo a passo completo para Windows, em PowerShell, na ordem certa. Os pré-requisitos (Build Tools do C++, WebView2 e Rust) conferi na documentação oficial do Tauri v2. Você não precisa instalar Node.js, porque a interface é HTML/JS puro.

Se tiver dúvida em qualquer etapa, pare e me mande a mensagem. Cada etapa tem um comando de verificação.

Antes de tudo: backup dos seus dados

Na primeira abertura, o programa une lojas repetidas que já estão no seu banco, e isso não tem volta. Copie a pasta:

powershell
if (Test-Path "$HOME\.prospector") { Copy-Item "$HOME\.prospector" "$HOME\.prospector-backup" -Recurse }
1. Ferramentas de C++ da Microsoft (obrigatório)

O Rust precisa delas para compilar. Pelo PowerShell:

powershell
winget --version
winget install --id Microsoft.VisualStudio.2022.BuildTools --override "--wait --passive --norestart --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"

Este comando winget com --override eu conheço de uso comum, mas não vem da documentação do Tauri. Se der erro ou o winget não existir, use o caminho oficial:

Baixe o instalador em https://visualstudio.microsoft.com/visual-cpp-build-tools/ e abra.
Marque “Desenvolvimento para desktop com C++” (em inglês, Desktop development with C++).
Clique em Instalar.

São vários GB e demora. Se o instalador pedir para reiniciar o computador, reinicie.

2. WebView2

O Windows 10 (versão 1803 ou mais nova) e o Windows 11 já trazem. Teste opcional:

powershell
Get-ItemProperty "HKLM:\SOFTWARE\WOW6432Node\Microsoft\EdgeUpdate\Clients\{F3017226-FE2A-4295-8BDF-00C3A9A7E4C5}" | Select-Object pv

Se aparecer um número de versão, está instalado. Se o comando falhar, baixe o “Evergreen Bootstrapper” na página do WebView2 Runtime da Microsoft e instale.

3. Rust
powershell
winget install --id Rustlang.Rustup --accept-package-agreements --accept-source-agreements

Se o winget não funcionar, baixe o instalador em https://rustup.rs. Feche o PowerShell e abra de novo (para o PATH atualizar). Depois:

powershell
rustup default stable-msvc
rustc --version
cargo --version

A documentação do Tauri recomenda a versão msvc no Windows. Os dois últimos comandos devem mostrar versões. Se disserem que não reconhecem rustc, feche e abra o PowerShell de novo, ou reinicie o computador.

4. Conferir o Python (você já tem)
powershell
python --version
5. Extrair o projeto novo

Use uma pasta nova, para não misturar com a versão anterior. Ajuste o caminho do zip se ele não estiver em Downloads:

powershell
Expand-Archive -Path "$HOME\Downloads\prospector-tauri.zip" -DestinationPath "$HOME\Documents\prospector-v11"
cd "$HOME\Documents\prospector-v11\prospector"
dir

Você deve ver build-desktop.bat, desktop, src, docs e outros.

6. Teste rápido, sem Rust (recomendo fazer antes)

Isso confirma que a parte Python e a interface funcionam no seu computador:

powershell
pip install -e .
python -m prospector.api --browser

O navegador abre com a interface nova. Faça uma pesquisa para testar. Para encerrar, feche a aba e espere cerca de 2 minutos, ou aperte Ctrl+C no PowerShell.

7. Gerar o instalador com Tauri
powershell
.\build-desktop.bat

Se o PowerShell bloquear scripts, use este equivalente:

powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\build-desktop.ps1

O script faz tudo em 6 etapas e mostra o andamento:

Cria o ambiente Python de construção.
Instala as dependências.
Roda os testes. Para pular: .\build-desktop.bat -SkipTests.
Empacota o motor Python (prospector-api.exe) e roda o autoteste dele.
Instala a ferramenta tauri-cli. Só na primeira vez, e demora alguns minutos, porque ela é compilada.
Compila o app. A primeira compilação do Rust também demora, e as próximas são bem mais rápidas.

No final aparece Pronto! Instalador: ...Prospector_1.1.0_x64-setup.exe. Se quiser fazer cada passo separadamente, depois de instalado o tauri-cli:

powershell
cargo install tauri-cli --version "^2" --locked
cargo tauri --version
8. Instalar e abrir

O instalador fica em:

powershell
dir .\desktop\src-tauri\target\release\bundle\nsis\
.\desktop\src-tauri\target\release\bundle\nsis\Prospector_1.1.0_x64-setup.exe

Ele instala só para o seu usuário, sem pedir administrador. Se aparecer “O Windows protegeu o computador”, clique em Mais informações e depois em Executar assim mesmo. É esperado, porque o instalador não tem assinatura digital paga, e foi você quem o gerou.

Depois, abra o Prospector pelo menu Iniciar. Na primeira abertura, ele une as repetidas do seu banco antigo.

Se algo der errado
Mensagem	O que fazer
rustc / cargo “não é reconhecido”	Feche e abra o PowerShell; se persistir, reinicie o computador
linker link.exe not found	A etapa 1 não foi concluída. Reabra o instalador do Visual Studio e confirme que Desktop development with C++ está marcado
winget não encontrado	Use os instaladores manuais das etapas 1 e 3
Script bloqueado	Use build-desktop.bat ou o comando com -ExecutionPolicy Bypass
Erro de compilação no main.rs	Eu não consegui compilar o Rust aqui. Copie a mensagem inteira e me envie que eu corrijo
Antivírus lento ou bloqueando	Libere a pasta do projeto e %USERPROFILE%\.cargo temporariamente
App abre mas diz “O motor não iniciou”	Veja C:\Users\<você>\.prospector\prospector.log e me envie as últimas linhas
Para desenvolver depois (opcional)

Para ver mudanças na interface sem gerar o instalador, entre em desktop e rode:

powershell
cd desktop
cargo tauri dev

Isso exige que o motor já esteja em desktop\src-tauri\binaries, o que o build-desktop.bat faz uma vez.

Comece pelas etapas 1 a 3 e me avise o resultado dos comandos de verificação (rustc --version e cargo --version) antes de seguir para o build.
