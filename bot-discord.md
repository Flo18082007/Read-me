# 🔐 Bot Discord avec MFA & Interface Web via Bun.js

## Ce projet permet de générer des codes secrets MFA via un bot Discord et de les vérifier sur une interface web propulsée par Bun.js.

## 📌 Prérequis
Node.js (optionnel si vous utilisez uniquement Bun)
Bun.js (alternative ultra-rapide à Node.js)
Discord Developer Account (pour créer un bot)
OTP Authenticator (Google Authenticator, Authy, etc.)

## 🛠️ Installation

### 1️⃣ Cloner le projet
```sh
git clone https://github.com/votre-repo/mfa-discord-bot.git
cd mfa-discord-bot
```

### 2️⃣ Installer Bun
```sh
curl -fsSL https://bun.sh/install | bash
```
### 3️⃣ Installer les dépendances
```sh
bun install
```
## 📝 Configuration

### 🔑 Créer un bot Discord
	1.	Aller sur Discord Developer Portal.
	2.	Créer une nouvelle application.
	3.	Ajouter un bot à l’application.
	4.	Copier le token du bot et le mettre dans .env.

### 🖥️ Configurer les variables d’environnement

#### Créer un fichier .env :
```sh
DISCORD_TOKEN=VOTRE_TOKEN_BOT
CLIENT_ID=VOTRE_CLIENT_ID
GUILD_ID=VOTRE_GUILD_ID
SECRET_KEY=une_clé_secrète_random
```
## 🚀 Démarrer le bot et le serveur

### Lancer le bot Discord et le serveur Web :
```sh
bun run start
bun run dev:web 
```
## 🏗️ Structure du projet
```sh
mfa-discord-bot/
│── src/
│   ├── bot.js        # Code du bot Discord
│   ├── server.js     # API backend avec Bun.js
│   ├── mfa.js        # Gestion MFA (OTP)
│── web/              # Interface web
│   ├── index.html
│   ├── app.js
│── .env              # Variables d'environnement
│── bun.lockb         # Fichier de verrouillage Bun
│── package.json      # Dépendances et scripts
│── README.md         # Documentation
```
## 🤖 Code du bot Discord

### Fichier : src/bot.js
```sh
import { Client, GatewayIntentBits } from "discord.js";
import { generateSecret, generateOTP } from "./mfa";
import "dotenv/config";

const client = new Client({
  intents: [GatewayIntentBits.Guilds, GatewayIntentBits.GuildMessages, GatewayIntentBits.MessageContent],
});

let userSecrets = {}; // Stockage temporaire des clés OTP

client.once("ready", () => {
  console.log(`✅ Bot connecté en tant que ${client.user.tag}`);
});

client.on("messageCreate", async (message) => {
  if (message.author.bot) return;

  if (message.content.startsWith("!mfa")) {
    const secret = generateSecret();
    userSecrets[message.author.id] = secret;

    message.author.send(`🔑 Votre clé secrète MFA : \`${secret}\`
Ajoutez-la dans votre application d'authentification.`);
  }
});

client.login(process.env.DISCORD_TOKEN);
```
## 🔑 Gestion MFA (OTP)

### Fichier : src/mfa.js
```sh
import { authenticator } from "otplib";

export function generateSecret() {
  return authenticator.generateSecret();
}

export function generateOTP(secret) {
  return authenticator.generate(secret);
}

export function verifyOTP(secret, token) {
  return authenticator.verify({ secret, token });
}
```
## 🌐 Backend Bun.js (API Web)

### Fichier : src/server.js
```sh
import { Elysia } from "elysia";
import { verifyOTP } from "./mfa";

const app = new Elysia();

app.post("/verify", async ({ body }) => {
  const { secret, token } = body;
  if (!secret || !token) return { success: false, error: "Données manquantes" };

  const isValid = verifyOTP(secret, token);
  return { success: isValid };
});

app.listen(3000, () => {
  console.log("🚀 Serveur Web lancé sur http://localhost:3000");
});
```
## 🎨 Interface Web (Frontend)

### Fichier : web/index.html
```sh
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vérification MFA</title>
    <script defer src="app.js"></script>
</head>
<body>
    <h1>🔐 Vérification MFA</h1>
    <input type="text" id="secret" placeholder="Votre clé secrète">
    <input type="text" id="token" placeholder="Code OTP">
    <button onclick="verify()">Vérifier</button>
    <p id="result"></p>
</body>
</html>
```
### Fichier : web/app.js
```sh
async function verify() {
    const secret = document.getElementById("secret").value;
    const token = document.getElementById("token").value;

    const response = await fetch("http://localhost:3000/verify", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ secret, token })
    });

    const data = await response.json();
    document.getElementById("result").innerText = data.success ? "✅ Code valide" : "❌ Code invalide";
}
```
## 📜 Commandes disponibles

Commande
Description
!mfa
Génère une clé secrète MFA et l’envoie en message privé

## 🛠️ Améliorations possibles
	•	🔄 Sauvegarde des clés dans une base de données (Redis, PostgreSQL…).
	•	📲 QR Code pour scanner plus facilement la clé MFA.
	•	🔔 Rappel automatique des OTP via Discord.

⸻

## 🏁 Conclusion

Ce projet démontre comment utiliser Bun.js pour gérer un bot Discord et une API web légère, permettant une authentification MFA sécurisée.