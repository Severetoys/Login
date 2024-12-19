<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Login Facial</title>
</head>
<body>
  <h1>Login Facial</h1>
  <video id="video" autoplay></video>
  <input type="text" id="nome" placeholder="Digite seu nome">
  <button id="register">Cadastrar</button>
  <button id="login">Entrar</button>
  <button id="capture" style="display: none;">Capturar</button> <!-- Este botão será ocultado até que o usuário escolha Cadastrar ou Entrar -->
  <canvas id="canvas" style="display: none;"></canvas>

  <script>
    const video = document.getElementById("video");
    const capture = document.getElementById("capture");
    const nomeInput = document.getElementById("nome");
    const registerButton = document.getElementById("register");
    const loginButton = document.getElementById("login");
    const API_URL = "https://isantos.cognitiveservices.azure.com/"; // Substitua com a URL da sua API
    const SHEET_URL = "https://script.google.com/macros/s/AKfycbw1k2_0n33QsNv8K1fDzgzdTWpaXeYr_K4hpRL2exyVWWV-l-V2foOsDBYRhdG4Sva8/exec"; // URL do Google Sheets

    // Ativa a câmera
    navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
      video.srcObject = stream;
      console.log("Câmera ativada com sucesso.");
    });

    // Cadastro de Usuário (Salvar rosto com nome)
    registerButton.addEventListener("click", () => {
      const nome = nomeInput.value;

      if (!nome) {
        alert("Por favor, insira seu nome para cadastro!");
        return;
      }

      capture.style.display = "inline"; // Mostra o botão de captura
      console.log("Cadastro iniciado para:", nome);
    });

    // Login de Usuário (Reconhecer rosto)
    loginButton.addEventListener("click", () => {
      const nome = nomeInput.value;

      if (!nome) {
        alert("Por favor, insira seu nome para login!");
        return;
      }

      capture.style.display = "inline"; // Mostra o botão de captura
      console.log("Login iniciado para:", nome);
    });

    // Captura a imagem e processa
    capture.addEventListener("click", () => {
      const context = canvas.getContext("2d");
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      context.drawImage(video, 0, 0, canvas.width, canvas.height);

      const imageData = canvas.toDataURL("image/png");
      const nome = nomeInput.value;

      if (!nome) {
        alert("Por favor, insira seu nome antes de capturar!");
        return;
      }

      console.log("Imagem capturada:", imageData);
      console.log("Nome fornecido:", nome);

      // Envia a imagem para a API de reconhecimento facial
      fetch(API_URL, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Ocp-Apim-Subscription-Key": "SUA_CHAVE_API" // Substitua pela sua chave
        },
        body: JSON.stringify({ image: imageData }),
      })
        .then((res) => res.json())
        .then((data) => {
          console.log("Resposta da API:", data);
          if (data.nomeReconhecido) {
            alert(`Bem-vindo, ${data.nomeReconhecido}!`);
          } else {
            alert("Rosto não reconhecido. Tente novamente.");
          }

          // Envia os dados para o Google Sheets
          fetch(SHEET_URL, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
              nome: nome,
              status: data.nomeReconhecido ? "Reconhecido" : "Não reconhecido"
            }),
          })
            .then((sheetRes) => console.log("Dados salvos no Google Sheets:", sheetRes))
            .catch((sheetErr) => console.error("Erro ao salvar no Sheets:", sheetErr));
        })
        .catch((err) => console.error("Erro ao processar imagem:", err));
    });
  </script>
</body>
</html>
