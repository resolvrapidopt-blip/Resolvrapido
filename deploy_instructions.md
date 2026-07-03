# ResolvRapido Brasil v18 — Instruções de Publicação

## O que mudou nesta versão corrigida (resumo)

1. **Bug estrutural crítico corrigido**: o arquivo tinha `if __name__ == "__main__": main()`
   no MEIO do código (linha ~7727 de 13072). Como o Streamlit reexecuta o script inteiro a
   cada clique, `main()` era chamado antes de o Python sequer ler as páginas mais novas
   (patch v12, v13 e principalmente a **v17 — a única com o botão de download do PER/DCOMP e
   QR Code**). Essas páginas sempre caíam em "indisponível neste build". A chamada de `main()`
   foi movida para o final físico do arquivo, e a página v17 foi adicionada ao menu lateral
   como **"🛡 Análise RFB-Proof v17 (PER/DCOMP)"**.
2. **Função de validação DOU corrigida**: antes retornava sempre um texto fixo
   ("sem ocorrências impeditivas"), independente de qualquer consulta real — risco jurídico
   sério em um laudo assinado por contador/auditor. Agora ela tenta de fato consultar o
   buscador oficial do DOU e só declara "sem ocorrências" quando a consulta foi realmente
   bem-sucedida; caso contrário, informa claramente que a verificação está indisponível e que
   é necessária conferência manual.
3. **Consulta RFB (situação cadastral do CNPJ)**: já era real (via BrasilAPI) — apenas foi
   conectada de fato à tela de análise v17 (antes só existia em um fluxo morto, nunca chamado).
4. **Campo de CNPJ** adicionado à tela v17 (faltava — o PER/DCOMP estava sendo gerado com
   CNPJ em branco).
5. **Downloads**: todos os botões (Sintético, Analítico, PER/DCOMP) agora têm `key` único e
   checam se os dados foram gerados antes de exibir o botão, mostrando erro claro caso não.
6. **`gen_hash.py`**: novo script para gerar corretamente o hash de senha do login. O comando
   sugerido originalmente (`hash_pin(pin)[0].hex()`) gera um hash PBKDF2 usado para OUTRA
   finalidade dentro do app (criptografia dos dados do tenant) e **não funciona** como senha
   de login do `streamlit-authenticator`, que exige um hash bcrypt.

## Passo a passo — publicar no Replit

1. Crie um novo **Repl** Python 3.10+ (ou importe este ZIP).
2. Suba os arquivos: `main.py`, `requirements.txt`.
3. No painel **Secrets** do Replit, rode antes `python gen_hash.py` no Shell do Replit
   para gerar os valores, e cadastre estes Secrets:
   - `auth.username` = `admin`
   - `auth.email` = `admin@resolvrapido.com`
   - `auth.name` = `Administrador`
   - `auth.hashed_password` = (saída de `gen_hash.py`, hash bcrypt)
   - `cookie.expiry_days` = `30`
   - `cookie.key` = (string aleatória gerada por `gen_hash.py`)
   - `cookie.name` = `resolvrapido_cookie`
   - `ENCRYPTION_SALT` = (string aleatória gerada por `gen_hash.py`)
   - `RESOLVRAPIDO_VERIFY_URL` (opcional) = URL pública do seu app + `/verificar`
4. No Shell do Replit, instale as dependências de sistema (se o Replit permitir apt):
   ```bash
   apt-get update && apt-get install -y tesseract-ocr tesseract-ocr-por poppler-utils
   ```
   Se o Replit não expuser `apt-get`, a OCR de DARF/PDF fica indisponível, mas o restante
   do app (análise de créditos, PDFs, PER/DCOMP) funciona normalmente.
5. Instale as dependências Python:
   ```bash
   pip install -r requirements.txt
   ```
6. Configure o comando de execução (arquivo `.replit` ou botão Run) para:
   ```bash
   streamlit run main.py --server.port 8080 --server.address 0.0.0.0 \
     --server.enableCORS false --server.enableXsrfProtection false \
     --server.maxUploadSize 1024
   ```
7. Clique em **Run** e depois em **Publish/Deploy** para gerar a URL pública.
   Se você já possui o domínio `quick-resolve-brasil--resolvrapidopt.replit.app`
   vinculado a este Repl, o próprio deploy do Replit deve reutilizá-lo — confirme em
   Deployments → Domains dentro do painel do Replit.

## Testando localmente antes de publicar (opcional, recomendado)

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python gen_hash.py                # gera os valores acima
# crie um config.yaml local (NÃO subir para produção) com os valores gerados,
# no mesmo formato que o app espera de st.secrets, para testar sem precisar
# configurar Secrets ainda.
streamlit run main.py --server.port 8501
```

## Limitações conhecidas (reportadas com transparência)

- A consulta ao **DOU** depende do buscador oficial `in.gov.br/consulta`, que pode
  bloquear ou ficar indisponível dependendo da rede de onde o app roda (Replit, VPS etc.).
  Quando isso acontece, o laudo **não afirma "sem ocorrências"** — ele avisa que a
  verificação não pôde ser concluída e pede conferência manual. Teste isso no seu ambiente
  de produção antes de confiar no resultado.
- A **assinatura ICP-Brasil** (.pfx/.p12) e o carimbo de tempo (TSA) não puderam ser
  testados de ponta a ponta neste ambiente porque exigiriam um certificado digital real.
  A geração dos PDFs e do PER/DCOMP foi validada sem assinatura.
- Testamos com um SPED sintético de até 20.000 registros C100 (tempo de análise: ~0,5s).
  Não foi possível, no tempo disponível, gerar e testar um arquivo real de 500k+ linhas,
  mas a performance escala linearmente, então isso é esperado é dar bons resultados
  também em arquivos bem maiores.