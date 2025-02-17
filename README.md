# Documentação Técnica: GitHub Feature/Release Candidate Deploy Workflow

Este workflow é responsável por realizar o processo completo de deploy de Feature Candidates e Release Candidates. Ele inclui:

•	Busca da última pré-release disponível no GitHub.

•	Configuração da AWS, incluindo credenciais e acesso a secrets.

•	Download e criptografia do .env para configurações sensíveis.

•	Autenticação e build da imagem Docker, com push para o Amazon ECR.

•	Criação de artefato de deploy contendo arquivos necessários para execução.

•	Conexão segura com instâncias EC2, incluindo VPN (OpenVPN) e SSH.

•	Execução de comandos remotos para implantar as mudanças.

•	Marcação da release como latest caso esteja na branch main.

•	Criação de Pull Request (PR) para merge da Release Candidate na main.

# Parâmetros de Entrada

**Credenciais AWS**

•	AWS_ACCESS_KEY_ID: Chave de acesso da AWS.

•	AWS_SECRET_ACCESS_KEY: Chave secreta da AWS.

•	AWS_REGION: Região da AWS onde os recursos estão alocados.

**Configuração do Ambiente**

•	AWS_S3_BUCKET: Nome do bucket S3 para baixar arquivos (opcional).

•	ENV_FILE_PATH: Caminho do arquivo de ambiente no S3 (opcional).

•	ENV_ENCRYPTION_PASSWORD: Senha para criptografia do arquivo .env (opcional).

•	SECRET_MANAGER_KEY: Chave para acessar secrets do AWS Secret Manager (opcional).

**Configuração Docker**

•	ECR_REGISTRY: URL do repositório ECR onde a imagem Docker será armazenada.

•	IMAGE_NAME: Nome da imagem Docker no ECR.

•	DOCKERFILE_PATH: Caminho para o Dockerfile utilizado na build.

•	BASE_IMAGE: Imagem base para a construção do container.

**Deploy para Servidores**

•	DEPLOY_FILES: Lista de arquivos necessários para o deploy.

•	SERVER_LIST: Lista de servidores para deploy (em formato JSON).

•	SSH_USER: Usuário SSH para acesso remoto.

•	SSH_KEY_FILE: Chave privada SSH para acesso às instâncias EC2.

**Conexão VPN**

•	OVPN_CONFIG_FILE: Arquivo de configuração OpenVPN (opcional).

•	OVPN_CERT_PASS: Senha para autenticação na VPN (opcional).

**Integração com GitHub**

•	GITHUB_TOKEN: Token do GitHub para operações na API.

# Etapas do Workflow

1. Checkout do Código
	•	Obtém os arquivos do repositório GitHub.

2. Busca da Última Pre-Release no GitHub
	•	Faz uma chamada à API do GitHub para encontrar a última release marcada como prerelease.
	•	Se nenhuma pré-release for encontrada, o workflow falha.

3. Configuração da AWS
	•	Autentica com a AWS utilizando as credenciais fornecidas.
	•	Se houver um arquivo .env no S3, ele será baixado e armazenado localmente.

4. Carregamento de Secrets
	•	Se for fornecida uma chave do AWS Secret Manager, os secrets são buscados e adicionados ao ambiente.

5. Criptografia do Arquivo .env
	•	Caso um password de criptografia seja informado, o arquivo .env será criptografado usando AES256.

6. Autenticação no ECR
	•	Se o repositório ECR for fornecido, o workflow realiza login no registro Docker da AWS.

7. Build e Push da Imagem Docker
	•	O workflow constrói a imagem Docker utilizando o Dockerfile especificado.
	•	A imagem recebe tags correspondentes à última pré-release encontrada no GitHub.
	•	Se o push estiver ativado, a imagem é enviada para o ECR.

8. Criação do Artefato de Deploy
	•	Todos os arquivos necessários para o deploy são empacotados em um .tar.gz e armazenados como artefato.

9. Busca de IPs das Instâncias EC2
	•	Obtém os endereços IP públicos das instâncias EC2 filtrando pelo nome informado.

10. Conexão VPN (se necessário)
	•	Se um arquivo de configuração OpenVPN for fornecido, a VPN será configurada e ativada.

11. Transferência do Artefato para EC2
	•	O artefato de deploy é transferido para as instâncias EC2 via SCP.

12. Execução Remota do Deploy
	•	O script de deploy é executado nas EC2 via SSH, realizando a atualização do ambiente.

13. Encerramento da VPN (se necessário)
	•	Caso a VPN tenha sido utilizada, ela será encerrada no final do processo.

14. Marcação da Release como Latest
	•	Se o workflow estiver rodando na branch main, a release será marcada como latest, removendo a flag prerelease.

15. Criação de PR para Merge da RC na main
	•	Se o workflow estiver rodando em uma branch rc/, será criado um Pull Request para merge da Release Candidate na main.

# Considerações

•	O deploy ocorre apenas para instâncias EC2 em estado running.

•	O workflow é flexível, permitindo deploys com ou sem Docker e com suporte a conexões VPN.

•	O uso do AWS Secret Manager permite uma gestão segura de credenciais e configurações sensíveis.

•	Se a branch for uma Release Candidate (rc/), um PR para main será criado automaticamente.