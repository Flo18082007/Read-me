🔐 Bot Discord avec MFA & Interface Web via Bun.js

Ce projet permet de générer des codes secrets MFA via un bot Discord et de les vérifier sur une interface web propulsée par Bun.js.

📌 Prérequis
	•	Node.js (optionnel si vous utilisez uniquement Bun)
	•	Bun.js (alternative ultra-rapide à Node.js)
	•	Discord Developer Account (pour créer un bot)
	•	OTP Authenticator (Google Authenticator, Authy, etc.)

🛠️ Installation

1️⃣ Cloner le projet

git clone https://github.com/votre-repo/mfa-discord-bot.git
cd mfa-discord-bot

2️⃣ Installer Bun

curl -fsSL https://bun.sh/install | bash

3️⃣ Installer les dépendances

bun install

📝 Configuration

🔑 Créer un bot Discord
	1.	Aller sur Discord Developer Portal.
	2.	Créer une nouvelle application.
	3.	Ajouter un bot à l’application.
	4.	Copier le token du bot et le mettre dans .env.

🖥️ Configurer les variables d’environnement

Créer un fichier .env :

DISCORD_TOKEN=VOTRE_TOKEN_BOT
CLIENT_ID=VOTRE_CLIENT_ID
GUILD_ID=VOTRE_GUILD_ID
SECRET_KEY=une_clé_secrète_random

🚀 Démarrer le bot et le serveur

Lancer le bot Discord et le serveur Web :

bun run start
bun run dev:web 

🏗️ Structure du projet

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

🤖 Code du bot Discord

Fichier : src/bot.js

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

🔑 Gestion MFA (OTP)

Fichier : src/mfa.js

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

🌐 Backend Bun.js (API Web)

Fichier : src/server.js

