// Requisitos:
// - Node.js
// - Puppeteer
// Instale com: npm install express puppeteer cors

const express = require('express');
const puppeteer = require('puppeteer');
const cors = require('cors');

const app = express();
const PORT = 5000;

app.use(cors());
app.use(express.json());

let urlLista = []; // Armazena os links enviados pelos usuários
let autoRespostaAtiva = true; // Controle da automação a cada 20 minutos

// Rota para receber links do front-end
app.post('/api/analyze', (req, res) => {
  const { url, auto } = req.body;
  if (!url) return res.status(400).json({ error: 'URL é obrigatória' });

  urlLista.push({ url, auto: auto ?? true });
  console.log(`🔗 Novo link adicionado: ${url} | Automação ativa: ${auto}`);
  res.status(200).json({ message: 'Link recebido. Será processado conforme a configuração.' });
});

// Rota para ativar/desativar automação global
app.post('/api/toggle-auto', (req, res) => {
  const { active } = req.body;
  autoRespostaAtiva = !!active;
  res.status(200).json({ message: `Automação global ${autoRespostaAtiva ? 'ativada' : 'desativada'}.` });
});

// Função que processa o link com Puppeteer
async function processarLink(url) {
  console.log(`🚀 Iniciando processamento: ${url}`);

  let browser;
  try {
    browser = await puppeteer.launch({ headless: true });
    const page = await browser.newPage();

    await page.goto(url, { waitUntil: 'networkidle2', timeout: 60000 });
    console.log(`🌐 Página carregada: ${url}`);

    // Espera por elementos de formulário dinâmico
    await page.waitForSelector('input, select, textarea', { timeout: 10000 });
    const inputs = await page.$$('input, select, textarea');

    console.log(`📋 Campos encontrados: ${inputs.length}`);

    // Preenchimento de exemplo
    for (const input of inputs) {
      const tipo = await input.evaluate(el => el.type);
      if (tipo === 'text') {
        await input.type('Resposta Automática');
      } else if (tipo === 'radio' || tipo === 'checkbox') {
        await input.click();
      }
    }

    // Clica no botão de envio
    const botao = await page.$('button[type="submit"], input[type="submit"]');
    if (botao) {
      await botao.click();
      console.log('📨 Formulário enviado automaticamente!');
    } else {
      console.log('⚠️ Nenhum botão de envio encontrado.');
    }

    await browser.close();
  } catch (error) {
    console.error('❌ Erro ao processar o link:', error.message);
    if (browser) await browser.close();
  }
}

// Executa a automação a cada 20 minutos (se ativada)
setInterval(() => {
  if (!autoRespostaAtiva) return;
  console.log('⏰ Iniciando varredura de links...');
  urlLista.forEach(({ url, auto }) => {
    if (auto) processarLink(url);
  });
}, 20 * 60 * 1000); // 20 minutos

// Início do servidor
app.listen(PORT, () => {
  console.log(`✅ Servidor iniciado na porta ${PORT}`);
  console.log('💻 Aguardando links para automatizar...');
});
