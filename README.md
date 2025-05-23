<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Mensagens Discord</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
            color: #333;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .warning {
            background-color: #fff3cd;
            border-left: 4px solid #ffc107;
            padding: 15px;
            margin-bottom: 20px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input, textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
        }
        textarea {
            height: 100px;
            resize: vertical;
        }
        button {
            background-color: #5865F2;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }
        button.stop {
            background-color: #f04747;
        }
        .status {
            margin-top: 15px;
            padding: 10px;
            border-radius: 4px;
        }
        .active {
            background-color: #d4edda;
            color: #155724;
        }
        .inactive {
            background-color: #f8d7da;
            color: #721c24;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Controle de Mensagens Discord</h1>
        
        <div class="warning">
            <h3>⚠️ Aviso Importante</h3>
            <p>Este site roda apenas no seu navegador. Seus dados NÃO são enviados para nenhum servidor externo.</p>
            <p>Nunca compartilhe seu token do Discord com ninguém. Ele dá acesso total à sua conta.</p>
            <p>Spam pode violar os Termos de Serviço do Discord e resultar em banimento.</p>
            <p>É recomendado que use o token de outra conta sua, para evitar que sua conta principal seja banida.</p>
        </div>

        <div class="form-group">
            <label for="token">Token do Discord:</label>
            <input type="password" id="token" placeholder="Insira seu token aqui">
        </div>

        <div class="form-group">
            <label for="serverId">ID do Servidor:</label>
            <input type="text" id="serverId" placeholder="ID do servidor alvo">
        </div>

        <div class="form-group">
            <label for="message">Mensagem:</label>
            <textarea id="message" placeholder="Digite a mensagem que será enviada"></textarea>
        </div>

        <div class="form-group">
            <label for="interval">Intervalo (segundos):</label>
            <input type="number" id="interval" value="5" min="1">
        </div>

        <button id="startBtn">Iniciar Envio</button>
        <button id="stopBtn" class="stop" disabled>Parar Envio</button>

        <div id="status" class="status inactive">
            Status: Inativo
        </div>
    </div>

    <script>
        let intervalId = null;
        let isRunning = false;
        let textChannels = []; // Armazenará os canais obtidos

        document.getElementById('startBtn').addEventListener('click', startSending);
        document.getElementById('stopBtn').addEventListener('click', stopSending);

        async function startSending() {
            const token = document.getElementById('token').value;
            const serverId = document.getElementById('serverId').value;
            const message = document.getElementById('message').value;
            const interval = document.getElementById('interval').value * 1000;

            if (!token || !serverId || !message) {
                alert('Preencha todos os campos!');
                return;
            }

            isRunning = true;
            updateUI();

            try {
                // Primeiro obtém todos os canais
                textChannels = await getChannels(token, serverId);
                
                // Envia imediatamente a primeira vez
                await sendMessagesToAll(token, message);
                
                // Configura o intervalo para envios subsequentes
                intervalId = setInterval(async () => {
                    await sendMessagesToAll(token, message);
                }, interval);
            } catch (error) {
                console.error('Erro ao iniciar envio:', error);
                stopSending();
                alert('Ocorreu um erro ao iniciar o envio.\n' + error.message);
            }
        }

        async function getChannels(token, serverId) {
            const response = await fetch(`https://discord.com/api/v9/guilds/${serverId}/channels`, {
                headers: { 'Authorization': token }
            });

            if (!response.ok) {
                throw new Error('Erro ao buscar canais');
            }

            const channels = await response.json();
            return channels.filter(ch => ch.type === 0); // Filtra apenas canais de texto
        }

        async function sendMessagesToAll(token, message) {
            if (textChannels.length === 0) {
                console.log('Nenhum canal disponível para enviar mensagens');
                return;
            }

            console.log(`Enviando mensagem para ${textChannels.length} canais...`);
            
            // Envia para todos os canais simultaneamente
            const sendPromises = textChannels.map(channel => {
                return fetch(`https://discord.com/api/v9/channels/${channel.id}/messages`, {
                    method: 'POST',
                    headers: {
                        'Authorization': token,
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ content: message })
                }).catch(err => {
                    console.error(`Erro ao enviar para ${channel.name}:`, err);
                    return null;
                });
            });

            await Promise.all(sendPromises);
            console.log('Mensagens enviadas para todos os canais');
        }

        function stopSending() {
            isRunning = false;
            clearInterval(intervalId);
            updateUI();
        }

        function updateUI() {
            const statusDiv = document.getElementById('status');
            const startBtn = document.getElementById('startBtn');
            const stopBtn = document.getElementById('stopBtn');

            if (isRunning) {
                statusDiv.textContent = 'Status: Ativo (enviando mensagens)';
                statusDiv.className = 'status active';
                startBtn.disabled = true;
                stopBtn.disabled = false;
            } else {
                statusDiv.textContent = 'Status: Inativo';
                statusDiv.className = 'status inactive';
                startBtn.disabled = false;
                stopBtn.disabled = true;
            }
        }
    </script>
</body>
</html>
