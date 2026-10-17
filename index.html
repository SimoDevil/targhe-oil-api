const express = require('express');
const { chromium } = require('playwright');
const fs = require('fs');
const path = require('path');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.static(__dirname));

const DB_FILE = path.join(__dirname, 'database_targhe.json');
const SESSION_FILE = path.join(__dirname, 'mpm_session.json');

const MPM_EMAIL = 'simonevarie2020@gmail.com';
const MPM_PASSWORD = 'Alessandra85';

if (!fs.existsSync(DB_FILE)) {
  fs.writeFileSync(DB_FILE, JSON.stringify({}, null, 2));
}

async function getAuthenticatedContext(browser) {
  if (fs.existsSync(SESSION_FILE)) {
    return await browser.newContext({
      storageState: SESSION_FILE,
      locale: 'it-IT'
    });
  }

  console.log('[LOGIN] Esecuzione login automatico su MPM Oil...');
  const context = await browser.newContext({ locale: 'it-IT' });
  const page = await context.newPage();

  try {
    await page.goto('https://www.mpmoil.it/login', { waitUntil: 'domcontentloaded' });
    
    const cookieBtn = page.locator('#CybotCookiebotDialogBodyLevelButtonLevelOptinAllowAll, button:has-text("Accetta")').first();
    if (await cookieBtn.isVisible({ timeout: 1500 }).catch(() => false)) {
      await cookieBtn.click();
    }

    await page.fill('input[type="email"], input[name*="email"], input[name*="username"]', MPM_EMAIL);
    await page.fill('input[type="password"], input[name*="password"]', MPM_PASSWORD);
    await page.click('button[type="submit"], input[type="submit"]');

    await page.waitForTimeout(3500);
    await context.storageState({ path: SESSION_FILE });
  } catch (err) {
    console.error('[LOGIN] Errore:', err);
  } finally {
    await page.close();
  }

  return context;
}

app.get('/api/targa/:plate', async (req, res) => {
  const plate = req.params.plate.replace(/\s+/g, '').toUpperCase();
  const db = JSON.parse(fs.readFileSync(DB_FILE, 'utf-8'));

  if (db[plate]) {
    return res.json({ source: 'Database Locale', data: db[plate] });
  }

  let browser;
  try {
    browser = await chromium.launch({ headless: true });
    const context = await getAuthenticatedContext(browser);
    const page = await context.newPage();

    await page.goto('https://www.mpmoil.it/products/recommendation', { waitUntil: 'domcontentloaded' });
    
    const plateInput = page.locator('input[type="text"], input[name*="plate"], input[name*="search"]').first();
    await plateInput.fill(plate);
    await plateInput.press('Enter');

    await page.waitForTimeout(3500);

    const currentUrl = page.url();
    if (!currentUrl.includes('/products/recommendation/1/')) {
      const firstResultLink = page.locator('a[href*="/products/recommendation/1/"]').first();
      if (await firstResultLink.isVisible({ timeout: 2500 }).catch(() => false)) {
        const href = await firstResultLink.getAttribute('href');
        const targetUrl = href.startsWith('http') ? href : `https://www.mpmoil.it${href}`;
        await page.goto(`${targetUrl}?searchtype=all`, { waitUntil: 'domcontentloaded' });
        await page.waitForTimeout(2500);
      }
    } else if (!currentUrl.includes('searchtype=all')) {
      await page.goto(`${currentUrl.split('?')[0]}?searchtype=all`, { waitUntil: 'domcontentloaded' });
      await page.waitForTimeout(2000);
    }

    const vehicleData = await page.evaluate(() => {
      const fullText = document.body.innerText;
      const headings = Array.from(document.querySelectorAll('h1, h2, h3'))
        .map(h => h.innerText.trim())
        .filter(t => t.length > 2 && !t.toLowerCase().includes('raccomandazioni'));

      const modelTitle = headings[0] || 'Veicolo Trovato';
      const modelSubtitle = headings[1] || '';
      const powerMatch = fullText.match(/(\d{2,3}\s*kW\s*(?:\/\s*\d{2,3}\s*(?:CV|HP))?)/i);
      const yearMatch = fullText.match(/\b((?:19|20)\d{2}\s*-\s*(?:(?:19|20)\d{2}|presente)?)\b/i);

      const components = [];
      const lines = fullText.split('\n').map(l => l.trim()).filter(Boolean);
      const sectionKeywords = ['Motore', 'Cambio', 'Trasmissione', 'Differenziale', 'Impianto frenante', 'Freni', 'Raffreddamento'];

      let currentComp = null;
      for (let i = 0; i < lines.length; i++) {
        const line = lines[i];
        const isHeader = sectionKeywords.some(kw => line.toLowerCase().startsWith(kw.toLowerCase())) && line.length < 65;

        if (isHeader) {
          if (currentComp && (currentComp.capacities.length > 0 || currentComp.specs.length > 0)) {
            components.push(currentComp);
          }
          currentComp = { name: line, capacities: [], intervals: [], viscosities: [], specs: [] };
          continue;
        }

        if (currentComp) {
          if (/capacit|litr|riempimento|filtro/i.test(line) && /\d/.test(line)) {
            currentComp.capacities.push(line);
          } else if (/\b\d+\s*(?:km|mesi|anni)\b/i.test(line) || /cambio:|controllo:/i.test(line)) {
            currentComp.intervals.push(line);
          }
          const viscMatch = line.match(/\b(\d{1,2}W-\d{2}|DOT\s*\d(?:\.\d)?|ATF|DCTF|G12\w*|G13)\b/i);
          if (viscMatch && !currentComp.viscosities.includes(viscMatch[1].toUpperCase())) {
            currentComp.viscosities.push(viscMatch[1].toUpperCase());
          }
          if (line.length > 3 && line.length < 120 && !/cookie|login|account/i.test(line)) {
            currentComp.specs.push(line);
          }
        }
      }
      if (currentComp && (currentComp.capacities.length > 0 || currentComp.specs.length > 0)) {
        components.push(currentComp);
      }

      return {
        modelTitle,
        modelSubtitle,
        power: powerMatch ? powerMatch[1] : 'Vedi dettagli',
        years: yearMatch ? yearMatch[1] : 'Vedi dettagli',
        sourceUrl: window.location.href,
        components: components.slice(0, 12),
        rawSummary: lines.slice(0, 140).join('\n')
      };
    });

    await browser.close();

    db[plate] = { ...vehicleData, savedAt: new Date().toLocaleString('it-IT') };
    fs.writeFileSync(DB_FILE, JSON.stringify(db, null, 2));

    return res.json({ source: 'Estratto Live', data: db[plate] });
  } catch (err) {
    if (browser) await browser.close();
    return res.status(500).json({ error: 'Errore durante la lettura.' });
  }
});

app.listen(process.env.PORT || 3000, () => {
  console.log('Server avviato!');
});
