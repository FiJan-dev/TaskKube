# Projeto 3 — TaskKube

Entrega prevista para **04/12/2026**. O projeto reaproveita o gerenciador de tarefas do Projeto 1. O GitHub Actions executará os testes. A demonstração do deployment será feita no Minikube local.

## 1. Preparar o ambiente

- [x] Centralizar a infraestrutura em um Vagrantfile com uma VM VirtualBox sem interface gráfica, 2 vCPUs e 3072 MiB de RAM.
- [x] Desativar a montagem de `/vagrant` e configurar o clone do repositório público por HTTPS em `/home/vagrant/TaskKube`. O clone usa a branch padrão do remoto. Em provisionamentos posteriores, a cópia existente é preservada.
- [x] Configurar Docker, Minikube com um nó de 1944 MiB e o complemento NGINX Ingress. A memória do nó sai da cota da VM.
- [x] Na primeira execução autorizada, conferir se há RAM disponível no host e se as VMs do Projeto 1 estão desligadas. Confirmar com `findmnt --mountpoint /vagrant` que não há montagem e com `git -C ~/TaskKube remote -v` que o clone aponta para o projeto.
  - [x] Conferir `minikube status` e `minikube kubectl -- get nodes`. Nas etapas abaixo, usar `minikube kubectl --` para executar comandos de kubectl dentro da VM.

## 2. Portar a aplicação do Projeto 1

- [x] Trazer `frontend/` (Next.js) e `backend/` (Express e Prisma) para o monorepositório TaskKube, incluindo os lockfiles e o schema. Excluir dependências instaladas, arquivos de ambiente locais e infraestrutura das três VMs antigas.
- [x] Criar um Dockerfile de produção por componente. Gerar o cliente Prisma e conferir o caminho de inicialização da API. Usar build em múltiplos estágios para o Next.js.
- [x] Configurar o workflow de publicação no GHCR a cada push na main, com uma imagem por componente e o SHA completo do commit como tag.
- [ ] Confirmar no Actions o primeiro build e a publicação de `ghcr.io/fijan-dev/taskkube-frontend:<sha>` e `ghcr.io/fijan-dev/taskkube-backend:<sha>`. Conferir se a VM tem acesso de leitura aos pacotes. Usar essas imagens no cluster para poupar a RAM que o build do Next.js consumiria na VM.
- [ ] Enviar o código ao remoto e atualizar o clone da VM com `git pull --ff-only` antes de aplicar os manifestos. Anotar o commit e as tags usados na demonstração.

## 3. Implantar e verificar o Kubernetes

- [ ] Criar um namespace para a aplicação e Services internos para frontend, API e PostgreSQL.
  - [ ] Implantar PostgreSQL com uma réplica e volume persistente. Fornecer credenciais por Secret e montar a URL do Prisma com o nome do Service do banco.
  - [ ] Aplicar o schema com uma etapa única e repetível antes de iniciar a API. Implantar uma réplica da API e verificar a prontidão em `/health`.
  - [ ] Implantar o frontend com `replicas: 3` e probe de prontidão. Definir requests e limites de CPU e memória por componente. Medir o consumo com `docker stats --no-stream` na VM e verificar eventos de pods pendentes ou encerrados por falta de memória.
- [ ] Criar Ingress de classe `nginx` com caminhos do tipo `Prefix`. Encaminhar `/api` para o Express preservando o caminho e `/` para o Service do frontend. O navegador já usa URLs relativas.
  - [ ] Para a demonstração, manter em execução na VM `minikube kubectl -- -n ingress-nginx port-forward --address=0.0.0.0 svc/ingress-nginx-controller 8080:80`. Abrir `http://127.0.0.1:8080` no host.
  - [ ] Conferir três endpoints prontos no Service do frontend. Fazer requisições sucessivas pelo Ingress e identificar nos logs do controlador os IPs dos pods que responderam.
  - [ ] Recriar o pod do banco e verificar se as tarefas persistem. Ao concluir a demonstração, desligar a VM com `vagrant halt` para liberar a RAM.

## 4. Implementar os testes

- [ ] **7 unitários** — isolar os handlers da API com dublês de Prisma e bcrypt. Escrever um teste independente por comportamento.
  - [ ] Cadastro com nome ausente retorna 400.
  - [ ] Cadastro com e-mail já usado retorna 400.
  - [ ] Login com senha ausente retorna 400.
  - [ ] Login com senha incorreta retorna 401.
  - [ ] Listagem sem identificador de usuário retorna 400.
  - [ ] Criação de tarefa sem título retorna 400.
  - [ ] Criação sem prioridade e sem status usa MEDIUM e PENDING.
- [ ] **3 de integração** — executar Express, Prisma e PostgreSQL de teste com schema aplicado e dados isolados por caso.
  - [ ] Cadastrar usuário pela API e efetuar login com a senha cadastrada.
  - [ ] Criar tarefas para dois usuários e verificar o filtro por usuário na listagem.
  - [ ] Editar e excluir uma tarefa pela API, conferindo o estado persistido após cada operação.
- [ ] **2 E2E** — usar Playwright com o frontend, a API e o banco reais, atrás de um único endereço HTTP.
  - [ ] Cadastrar e autenticar um usuário pelo navegador e verificar a página de tarefas.
  - [ ] Com um usuário de teste, criar, editar e excluir uma tarefa pela interface, verificando a lista após recarregar a página.

## 5. Automatizar e apresentar

- [ ] Criar workflow em `.github/workflows/` com `push` na `main` e `pull_request` com destino à `main`. Executar as três suítes em runner hospedado pelo GitHub.
  - [ ] Instalar as dependências pelos lockfiles, preparar o banco de teste e aguardar a prontidão dos serviços antes dos testes.
  - [ ] Fazer o workflow falhar quando uma suíte falhar e guardar relatórios dos testes. Condicionar a publicação das imagens ao sucesso dos testes na main. O deployment pode ser manual.
  - [ ] Registrar uma execução de PR e uma execução da main com os resultados das três suítes.
- [ ] Preparar a avaliação com o commit usado, evidências dos cinco pods da aplicação, balanceamento pelo Ingress e histórico do Actions. Se a entrega ocorrer pelo Canvas, seguir a orientação do professor sobre o checksum.
