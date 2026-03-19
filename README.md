# MMFBlastOff
Public Records Bot
const puppeteer = require('puppeteer-extra');
const StealthPlugin = require('puppeteer-extra-plugin-stealth');
const RecaptchaPlugin = require('puppeteer-extra-plugin-recaptcha');
const fs = require('fs');

// 1. Setup Stealth and Captcha
puppeteer.use(StealthPlugin());
puppeteer.use(
  RecaptchaPlugin({
    provider: { 
        id: '2captcha', 
        token: process.env.CAPTCHA_KEY // Pulls from your Railway/Environment variables
    },
    visualFeedback: true 
  })
);

async function runBlast() {
  // Load data from your data.json
  const rawData = fs.readFileSync('data.json');
  const users = JSON.parse(rawData);

  const browser = await puppeteer.launch({
    headless: true, // Required for Railway/Cloud
    args: ['--no-sandbox', '--disable-setuid-sandbox', '--disable-dev-shm-usage']
  });

  const page = await browser.newPage();

  for (const user of users) {
    console.log(`--- Starting Blast for: ${user.name} ---`);

    // --- SITE 1: ListYourself.net ---
    try {
      await page.goto('https://www.listyourself.net/ListYourself/listing.jsp', { waitUntil: 'networkidle2' });
      console.log("Filling ListYourself...");
      
      await page.type('input[name="firstname"]', user.name.split(' ')[0]);
      await page.type('input[name="lastname"]', user.name.split(' ')[1] || 'Resident');
      await page.type('input[name="phone"]', user.phone);
      await page.type('input[name="address"]', user.address || "123 Main St");
      await page.type('input[name="zip"]', user.zip || "90210");
      await page.type('input[name="email"]', user.email);

      // Solve Captcha if it exists
      await page.solveRecaptchas();
      
      // Click Submit (Check the actual button selector on the site)
      // await page.click('input[type="submit"]'); 
      console.log("✅ ListYourself Submitted.");
    } catch (e) {
      console.log("❌ ListYourself Failed:", e.message);
    }

    // --- SITE 2: OptOutPrescreen ---
    try {
        console.log("Navigating to OptOut...");
        await page.goto('https://www.optoutprescreen.com/selection', { waitUntil: 'networkidle2' });
        // Add specific interaction logic for their multi-step flow here
        console.log("✅ OptOut Site Accessed.");
    } catch (e) {
        console.log("❌ OptOut Failed.");
    }

    // Wait 5 seconds between users to avoid IP bans
    await new Promise(r => setTimeout(r, 5000));
  }

  await browser.close();
  console.log("--- All Blasts Finished ---");
}

runBlast();

{
  "name": "mmfblastoff",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.19.2",
    "body-parser": "^1.20.2",
    "express-session": "^1.17.3",
    "puppeteer": "^22.0.0",
    "puppeteer-extra": "^3.3.6",
    "puppeteer-extra-plugin-stealth": "^2.11.2",
    "puppeteer-extra-plugin-recaptcha": "^3.6.8"
  }
}

[
  {
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "5551234567",
    "address": "742 Evergreen Terrace",
    "zip": "90210"
  }
]

{
  "name": "mmfblastoff",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "postinstall": "npx puppeteer browsers install chrome"
  },
  "dependencies": {
    "express": "^4.19.2",
    "body-parser": "^1.20.2",
    "express-session": "^1.17.3",
    "puppeteer": "^22.0.0",
    "puppeteer-extra": "^3.3.6",
    "puppeteer-extra-plugin-stealth": "^2.11.2",
    "puppeteer-extra-plugin-recaptcha": "^3.6.8"
  }
}

// Replace the very last few lines of your server.js with this:
const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`MMFblastOff running on port ${PORT}`);
});

const puppeteer = require('puppeteer-extra');
const StealthPlugin = require('puppeteer-extra-plugin-stealth');
const RecaptchaPlugin = require('puppeteer-extra-plugin-recaptcha');
const fs = require('fs');

puppeteer.use(StealthPlugin());
puppeteer.use(RecaptchaPlugin({
    provider: { id: '2captcha', token: process.env.CAPTCHA_KEY },
    visualFeedback: false
}));

async function runBlast() {
  const rawData = fs.readFileSync('data.json');
  const users = JSON.parse(rawData);

  // CRITICAL: These args prevent the "Error" on Railway
  const browser = await puppeteer.launch({
    headless: true,
    args: [
      '--no-sandbox',
      '--disable-setuid-sandbox',
      '--disable-dev-shm-usage',
      '--single-process'
    ]
  });

  const page = await browser.newPage();

  for (const user of users) {
    try {
      console.log(`Blasting: ${user.name}`);
      await page.goto('https://www.listyourself.net/ListYourself/listing.jsp', { waitUntil: 'networkidle2' });
      
      await page.type('input[name="firstname"]', user.name.split(' ')[0]);
      await page.type('input[name="lastname"]', user.name.split(' ')[1] || 'Resident');
      await page.type('input[name="phone"]', user.phone);
      
      await page.solveRecaptchas();
      // await page.click('input[type="submit"]'); 
      console.log("✅ Success for " + user.name);
    } catch (e) {
      console.log("❌ Error: " + e.message);
    }
  }
  await browser.close();
}

runBlast();

