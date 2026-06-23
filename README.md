# App 3fácil — Cliente Final

App mobile (iOS/Android) para que o cliente final acompanhe os anúncios da sua
corretora e o histórico de atendimento, sincronizado com o CRM próprio do
3fácil (banco MySQL).

## Estrutura

```
3facil-app/
  backend/     -> API Node.js que lê do banco MySQL do CRM
  mobile/      -> App React Native (Expo) usado pelo cliente final
```

O app **nunca** acessa o banco de dados diretamente — ele conversa só com a
API do backend, que por sua vez lê do MySQL do CRM. Isso evita expor
credenciais do banco no celular do cliente e protege o CRM de sobrecarga.

## Passo 1 — Backend

```bash
cd backend
npm install
cp .env.example .env
# edite o .env com host/usuário/senha reais do banco MySQL do CRM
npm run dev
```

A API sobe em `http://localhost:3001`. Teste com:
```bash
curl http://localhost:3001/api/health
```

### ⚠️ Antes de rodar de verdade, ajuste:
1. **Credenciais do banco** em `.env` — idealmente crie um usuário MySQL
   *somente leitura* (`GRANT SELECT`) para o app, separado do usuário que o
   CRM usa para escrever.
2. **Nomes de tabelas/colunas** nos arquivos em `backend/src/routes/*.js`.
   Usei os nomes `imoveis`, `leads`, `propostas`, `clientes`, `corretores`,
   `favoritos` com base nas telas do CRM que você enviou — confirme com quem
   mantém o banco se são esses os nomes reais.
3. **Tabela `favoritos`**: provavelmente não existe ainda no CRM (é uma
   funcionalidade nova do app). Vai precisar criar essa tabela:
   ```sql
   CREATE TABLE favoritos (
     id INT AUTO_INCREMENT PRIMARY KEY,
     cliente_id INT NOT NULL,
     imovel_id INT NOT NULL,
     criado_em DATETIME DEFAULT NOW(),
     UNIQUE KEY uniq_cliente_imovel (cliente_id, imovel_id)
   );
   ```

## Passo 2 — App mobile

```bash
cd mobile
npm install
npx expo start
```

Escaneie o QR code com o app **Expo Go** (Android/iOS) para testar no celular
de verdade, sem precisar publicar nada ainda.

### ⚠️ Antes de rodar:
- Em `mobile/src/api.js`, troque `API_URL` pelo IP da sua máquina na rede
  local (não use `localhost` — o celular não acessa o localhost do
  computador). Exemplo: `http://192.168.0.10:3001/api`.
- Quando for para produção, troque pela URL pública do backend (ex: depois de
  publicar em um servidor/Render/Railway/EC2).

## Próximos passos sugeridos
- Validar com você a estrutura real das tabelas do MySQL e ajustar as queries
- Implementar push notifications reais (Expo Notifications) para avisos de
  novo imóvel / mudança de status
- Publicar a API em um servidor (mesma rede do banco do CRM, por segurança)
- Gerar o build do app para a loja (`eas build`) quando estiver validado
