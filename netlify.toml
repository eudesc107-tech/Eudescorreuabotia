/**
 * BOT COMENTE E RECEBA NO DIRECT - Instagram
 * Perfil: @eudescorreia
 * Versao para Netlify Functions (via GitHub)
 *
 * ESTRUTURA DO REPOSITORIO NO GITHUB:
 * seu-repositorio/
 *   netlify.toml
 *   netlify/
 *     functions/
 *       webhook.js   (este arquivo)
 *
 * VARIAVEIS DE AMBIENTE (Netlify > Site configuration > Environment variables):
 *   VERIFY_TOKEN  = uma senha nova que voce inventa
 *   ACCESS_TOKEN  = token gerado no app da Meta
 *   IG_USER_ID    = ID numerico da conta @eudescorreia
 *   TRIGGER_WORDS = LINK,QUERO,IA,GPT
 *
 * APOS O DEPLOY, sua funcao fica em:
 *   https://SEU-SITE.netlify.app/.netlify/functions/webhook
 *
 * Para ativar: .../webhook?action=setup
 * Para conferir: .../webhook?action=status
 */

// ===== MENSAGEM DA DM =====
const DM_MESSAGE =
  "Oi! Vi que voce comentou no meu video. " +
  "Aqui esta o link de acesso a todos os comandos que eu uso nos videos, " +
  "aproveita: https://tinyurl.com/mvhkc7w";

// Resposta publica opcional no proprio comentario (deixe "" para desativar)
const PUBLIC_REPLY = "Te chamei no Direct!";

exports.handler = async (event) => {
  const VERIFY_TOKEN = process.env.VERIFY_TOKEN || "";
  const ACCESS_TOKEN = process.env.ACCESS_TOKEN || "";
  const IG_USER_ID = process.env.IG_USER_ID || "";
  const TRIGGER_WORDS = process.env.TRIGGER_WORDS || "LINK,QUERO,IA,GPT";

  const params = event.queryStringParameters || {};

  // ----- ROTA DE ATIVACAO: acesse ?action=setup uma unica vez -----
  if (event.httpMethod === "GET" && params.action === "setup") {
    const res = await fetch(
      `https://graph.instagram.com/v23.0/${IG_USER_ID}/subscribed_apps?subscribed_fields=comments,messages`,
      {
        method: "POST",
        headers: { Authorization: `Bearer ${ACCESS_TOKEN}` },
      },
    );
    const data = await res.json();
    return {
      statusCode: 200,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data, null, 2),
    };
  }

  // ----- ROTA DE DIAGNOSTICO: acesse ?action=status para conferir -----
  if (event.httpMethod === "GET" && params.action === "status") {
    const res = await fetch(
      `https://graph.instagram.com/v23.0/${IG_USER_ID}/subscribed_apps`,
      {
        headers: { Authorization: `Bearer ${ACCESS_TOKEN}` },
      },
    );
    const data = await res.json();
    return {
      statusCode: 200,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data, null, 2),
    };
  }

  // ----- 1. VERIFICACAO DO WEBHOOK (a Meta faz isso uma vez) -----
  if (event.httpMethod === "GET") {
    const mode = params["hub.mode"];
    const token = params["hub.verify_token"];
    const challenge = params["hub.challenge"];

    if (mode === "subscribe" && token === VERIFY_TOKEN) {
      return { statusCode: 200, body: challenge };
    }
    return { statusCode: 200, body: "Bot eudescorreia ativo" };
  }

  // ----- 2. RECEBENDO EVENTOS (comentarios chegam aqui) -----
  if (event.httpMethod === "POST") {
    let body;
    try {
      body = JSON.parse(event.body);
    } catch {
      return { statusCode: 200, body: "OK" };
    }

    console.log("Evento recebido:", JSON.stringify(body));

    try {
      for (const entry of body.entry || []) {
        for (const change of entry.changes || []) {
          if (change.field !== "comments") continue;

          const comment = change.value;
          const commentId = comment.id;
          const commentText = (comment.text || "").trim();
          const authorId = comment.from ? comment.from.id : null;

          // Ignora comentarios da propria conta (evita loop infinito)
          if (authorId && authorId === IG_USER_ID) continue;

          // Verifica se o comentario contem alguma palavra gatilho
          const triggers = TRIGGER_WORDS.split(",")
            .map((w) => w.trim().toLowerCase())
            .filter(Boolean);

          const textLower = commentText.toLowerCase();
          const matched = triggers.some((t) => textLower.includes(t));

          if (!matched) continue;

          // Envia a DM privada em resposta ao comentario
          await sendPrivateReply(IG_USER_ID, ACCESS_TOKEN, commentId);

          // Responde publicamente no comentario (opcional)
          if (PUBLIC_REPLY) {
            await sendPublicReply(ACCESS_TOKEN, commentId);
          }
        }
      }
    } catch (err) {
      console.log("Erro ao processar evento:", err.message);
    }

    return { statusCode: 200, body: "EVENT_RECEIVED" };
  }

  return { statusCode: 200, body: "Bot eudescorreia ativo" };
};

// Envia mensagem privada no Direct em resposta a um comentario
async function sendPrivateReply(igUserId, accessToken, commentId) {
  const res = await fetch(
    `https://graph.instagram.com/v23.0/${igUserId}/messages`,
    {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${accessToken}`,
      },
      body: JSON.stringify({
        recipient: { comment_id: commentId },
        message: { text: DM_MESSAGE },
      }),
    },
  );
  const data = await res.json();
  console.log("Resposta DM:", JSON.stringify(data));
}

// Responde publicamente no comentario
async function sendPublicReply(accessToken, commentId) {
  const res = await fetch(
    `https://graph.instagram.com/v23.0/${commentId}/replies`,
    {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${accessToken}`,
      },
      body: JSON.stringify({ message: PUBLIC_REPLY }),
    },
  );
  const data = await res.json();
  console.log("Resposta publica:", JSON.stringify(data));
}
