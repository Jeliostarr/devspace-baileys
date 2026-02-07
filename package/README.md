![Banner](https://capsule-render.vercel.app/api?type=waving&color=0091FF&height=220&section=header&text=@arceos/baileys&fontSize=90&animation=fadeIn&fontAlignY=35&desc=High-Performance%20WhatsApp%20Web%20API%20Library&descAlignY=55&descAlign=50)

---

[![npm version](https://img.shields.io/npm/v/@arceos/baileys.svg?style=for-the-badge&logo=npm&color=CB3837)](https://www.npmjs.com/package/@arceos/baileys)
[![downloads](https://img.shields.io/npm/dt/@arceos/baileys.svg?style=for-the-badge&logo=npm&color=CB3837)](https://www.npmjs.com/package/@arceos/baileys)
[![license](https://img.shields.io/npm/l/@arceos/baileys.svg?style=for-the-badge&color=green)](LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.5+-3178C6?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)

---

**The most advanced, lightweight, and optimized fork of Baileys.** Built for speed, stability, and modern WhatsApp features without heavy dependencies.

[Features](#-key-features) •
[Installation](#-installation) •
[Quick Start](#-quick-start) •
[Documentation](#-documentation) •
[Credits](#-credits)

---

## 📖 Introduction

@arceos/baileys is a reimagined version of the classic WhatsApp Web API library. Unlike other forks, this version is meticulously optimized for **Node.js** environments, removing heavy image processing dependencies (Jimp) in favor of native system calls (FFmpeg) and lightweight buffers.

It supports the latest Multi-Device architecture, ensuring your bot stays connected longer, responds faster, and consumes less memory.

## Why use @arceos/baileys?  

| Feature | @arceos/baileys | Standard Baileys |
| :--- | :---: | :---: |
| **Speed** | 🚀 **Extreme (95% Faster)** | Standard |
| **Dependencies** | 🪶 **Lightweight (No Jimp)** | Heavy |
| **Platform Support** | 📱 **Android, iOS, Web** | Mostly Web/Android |

---

## 💡 Key Features


* ⚡ **Zero-Bloat Architecture:** Removed `Jimp` and other heavy dependencies. Uses native streams and buffers.
* 🔐 **Enterprise Security:** Full End-to-End Encryption (E2EE) support identical to the official WhatsApp client.
* 📱 **Custom Pairing Code:** Pairing Codes can now be customized to make them easier to read and more professional.
* 📂 **Multi-Session Storage:** Built-in support for `useMultiFileAuthState` to save sessions locally without data loss.
* 🛡️ **Other Features:**
    * **iOS & Apple Support:** Full support for iOS and Apple devices! Special optimizations for Safari and macOS.
    * **Auto Recovery:** The bot keeps running even if the connection is lost! Automatically logs in from the last session without re-scanning the QR code.
    * **Anti-Call:** Auto-reject incoming calls to prevent connection drops.
    * **Anti-Spam:** Protects your number from being flagged as spam.
* 📨 **Advanced Messaging:**
    * Send **Polls** (Voting).
    * Send **Albums** (Grouped media).
    * Send **Interactive Buttons & Lists**.
    * **Dll**.

---

## 📋 Requirements
* **Node.js: >= ``20.0.0``**
* **OS: ``Windows | Linux | macOS``**
* **Memory: >= ``1GB RAM``**
* **TypeScript: >= ``5.5+ (optional)``**

---

## 📦 Installation

Ensure you have **Node.js v20.0.0** or higher and **FFmpeg** installed on your system.

```bash
Install via NPM
npm install @arceos/baileys

Install via Yarn
yarn add @arceos/baileys

Install via PNPM
pnpm add @arceos/baileys
```

---

## 🚀 Quick Start

**1. Basic Connection (QR Code)**

* Create a file named index.js and add the following code:

```javascript
import { makeWASocket, DisconnectReason, useMultiFileAuthState } from "@arceos/baileys"
import { Boom } from "@hapi/boom"
import P from "pino"

async function startBot() {
    const { state, saveCreds } = await useMultiFileAuthState("session_auth")

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true,
        logger: P({ level: "silent" }),
        browser: ["Linux", "Chrome", "20.0.04"],
        connectTimeoutMs: 60000,
        defaultQueryTimeoutMs: 0,
        keepAliveIntervalMs: 10000,
        emitOwnEvents: true,
        fireInitQueries: true,
        generateHighQualityLinkPreview: true,
        syncFullHistory: true,
        markOnlineOnConnect: true,
    })

    sock.ev.on("connection.update", (update) => {
        const { connection, lastDisconnect } = update
        if (connection === "close") {
            const shouldReconnect = (lastDisconnect.error)?.output?.statusCode !== DisconnectReason.loggedOut
            if (shouldReconnect) {
                startBot()
            }
        } else if (connection === "open") {
            console.log("Connected to WhatsApp Successfully")
        }
    })

    sock.ev.on("creds.update", saveCreds)

    sock.ev.on("messages.upsert", async ({ messages }) => {
        const m = messages[0]
        if (!m.message) return
        console.log(JSON.stringify(m, undefined, 2))
    })
}

startBot()
```

**2. Connection via Pairing Code (No QR)**

* If you prefer using a Pairing Code instead of scanning a QR code:

```javascript
import { makeWASocket, useMultiFileAuthState, delay } from "@arceos/baileys"
import P from "pino"

async function startPairing() {
    const { state, saveCreds } = await useMultiFileAuthState("session_pairing")

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: false,
        logger: P({ level: "silent" }),
        browser: ["Linux", "Chrome", "20.0.04"]
    })

    if (!sock.authState.creds.registered) {
        const phoneNumber = "6281234567890" 
        await delay(3000)
        const code = await sock.requestPairingCode(phoneNumber)
        console.log(`Your Pairing Code: ${code}`)
    }

    sock.ev.on("creds.update", saveCreds)
}

startPairing()
```

----

## 📚 Documentation

#### Text & Media Message

```javascript
await sock.sendMessage("62812345678@s.whatsapp.net", { 
    text: "Hello World from @arceos/baileys!" 
})

Image
await sock.sendMessage(jid, { 
    image: { url: "[https://example.com/pic.jpg](https://example.com/pic.jpg)" }, 
    caption: "Here is an image" 
})

Video
await sock.sendMessage(jid, { 
    video: { url: "./local_video.mp4" }, 
    caption: "Here is a video",
    gifPlayback: true 
})

Audio
await sock.sendMessage(jid, { 
    audio: { url: "./audio.mp3" }, 
    mimetype: "audio/mp4",
    ptt: true 
})

Polls (Voting)
await sock.sendMessage(jid, {
    poll: {
        name: "Which is better?",
        values: ["@arceos/baileys", "Original Baileys"],
        selectableCount: 1
    }
})
```

##### Send Album

```javascript
await sock.sendAlbumMessage(jid, [
    { image: { url: "./img1.jpg" }, caption: "Photo 1" },
    { image: { url: "./img2.jpg" }, caption: "Photo 2" },
    { video: { url: "./vid1.mp4" }, caption: "Video 1" }
], { quoted: m })
```

---

#### Interactive Buttons

Note: Button support depends on the WhatsApp version and region.

```javascript
await sock.sendMessage(jid, {
    text: "Please select an option below:",
    footer: "Arceo Bot System",
    buttons: [
        { buttonId: "id1", buttonText: { displayText: "Buy Premium" }, type: 1 },
        { buttonId: "id2", buttonText: { displayText: "Contact Support" }, type: 1 }
    ],
    headerType: 1
})
```
##### Button List

```javascript
const sections = [
    {
        title: "Main Menu",
        rows: [
            { title: "Option 1", rowId: "opt1", description: "Description for option 1" },
            { title: "Option 2", rowId: "opt2", description: "Description for option 2" }
        ]
    }
]

await sock.sendMessage(jid, {
    text: "Open the menu",
    buttonText: "Click Here",
    sections
})
```

---

🙏 Credits
This library is built upon the hard work of the open-source community.
 * Original Core: WhiskeySockets/Baileys
 * Base Optimizations: @realvare/based
 * Performance Engineering: Arceo

Maintained with ❤️ by Arceo

Licensed under MIT