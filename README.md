
export const handler = async (event) => {
  if (event.httpMethod !== "POST") {
    return {
      statusCode: 405,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ error: "Use o método POST." }),
    };
  }

  const apiKey = process.env.GROQ_API_KEY;

  if (!apiKey) {
    return {
      statusCode: 500,
      body: JSON.stringify({
        error: "A chave GROQ_API_KEY não está configurada no Netlify.",
      }),
    };
  }

  try {
    const body = JSON.parse(event.body || "{}");
    const messages = body.messages;

    if (
      !Array.isArray(messages) ||
      messages.length === 0 ||
      !messages.every(
        (m) =>
          m &&
          ["user", "assistant"].includes(m.role) &&
          typeof m.content === "string"
      )
    ) {
      return {
        statusCode: 400,
        body: JSON.stringify({
          error: "A conversa enviada é inválida.",
        }),
      };
    }

    const response = await fetch(
      "https://api.groq.com/openai/v1/chat/completions",
      {
        method: "POST",
        headers: {
          Authorization: `Bearer ${apiKey}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          model: "llama-3.3-70b-versatile",
          messages: [
            {
              role: "system",
              content:
                "Você é o ScriptForgeAI, um assistente especializado em Roblox Studio e Luau. Ajude o usuário a entender o jogo informado e a criar scripts claros, comentados e adequados ao contexto. Não afirme que pesquisou o jogo na internet se não tiver feito isso. Priorize scripts para jogos próprios ou testes autorizados. Explique onde colocar o código e como testar.",
            },
            ...messages,
          ],
          temperature: 0.7,
          max_tokens: 4000,
        }),
      }
    );

    const data = await response.json();

    if (!response.ok) {
      console.error("Erro da API Groq:", response.status);

      return {
        statusCode: response.status === 429 ? 429 : 502,
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          error:
            response.status === 401
              ? "A chave da Groq é inválida. Confira a variável no Netlify."
              : response.status === 429
                ? "Limite de uso da Groq atingido. Tente novamente mais tarde."
                : "A Groq não conseguiu processar o pedido.",
        }),
      };
    }

    const answer = data.choices?.[0]?.message?.content;

    if (!answer) {
      return {
        statusCode: 502,
        body: JSON.stringify({
          error: "A Groq retornou uma resposta vazia.",
        }),
      };
    }

    return {
      statusCode: 200,
      headers: {
        "Content-Type": "application/json; charset=utf-8",
      },
      body: JSON.stringify({ answer }),
    };
  } catch (error) {
    console.error("Erro na função generate:", error.message);

    return {
      statusCode: 500,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        error: "Erro interno ao gerar a resposta.",
      }),
    };
  }
};
