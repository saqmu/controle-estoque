<!DOCTYPE html>
<html>
<head>
    <title>Leitor de Estoque</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        body { font-family: Arial, sans-serif; max-width: 500px; margin: auto; padding: 20px; }
        #qr-reader { width: 100%; border: 2px solid #ccc; }
        #form-container { display: none; margin-top: 20px; }
        input, select, button { width: 100%; padding: 10px; margin-top: 10px; box-sizing: border-box; font-size: 16px; }
        button { background-color: #007bff; color: white; border: none; cursor: pointer; }
        #success-message, #error-message { padding: 15px; margin-top: 10px; border-radius: 5px; text-align: center; }
        #success-message { background-color: #d4edda; color: #155724; }
        #error-message { background-color: #f8d7da; color: #721c24; }
    </style>
</head>
<body>
    <h1>Controle de Almoxarifado</h1>

    <!-- Onde a câmera vai aparecer -->
    <div id="qr-reader"></div>

    <!-- O formulário que aparece depois da leitura -->
    <div id="form-container">
        <h2>Produto Lido</h2>
        <input type="text" id="codigo" name="codigo" readonly>
        
        <select id="tipo" name="tipo">
            <option value="Saída do Estoque">Saída do Estoque</option>
            <option value="Entrada no Estoque">Entrada no Estoque</option>
        </select>
        
        <input type="number" id="quantidade" name="quantidade" placeholder="Quantidade" required>
        <input type="text" id="nome" name="nome" placeholder="Seu Nome" required>
        
        <button onclick="submitForm()">Registrar Movimentação</button>
        
        <div id="success-message" style="display:none;"></div>
        <div id="error-message" style="display:none;"></div>
    </div>

    <!-- A biblioteca mágica que lê QR Code -->
    <script src="https://unpkg.com/html5-qrcode/html5-qrcode.min.js"></script>

    <script>
        // <<< COLE A URL DA SUA PONTE DO APPS SCRIPT AQUI DENTRO DAS ASPAS >>>
        const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbzD3EolMrY4fvDj7aicQKpNGzAdQ6XOL_DEugKO9g7LJ9eARe7FRmKopnfeb_wQLAvdCA/exec" 

        const qrReaderDiv = document.getElementById('qr-reader');
        const formContainer = document.getElementById('form-container');

        // Função que é chamada quando o QR code é lido com sucesso
        function onScanSuccess(decodedText, decodedResult) {
            console.log(`Code matched = ${decodedText}`, decodedResult);
            
            // Para o leitor de QR Code
            html5QrcodeScanner.clear();
            qrReaderDiv.style.display = 'none';

            // Preenche o campo de código e mostra o formulário
            document.getElementById('codigo').value = decodedText;
            formContainer.style.display = 'block';
            
            alert(`Produto Lido: ${decodedText}`);
        }

        // Configuração do leitor de QR Code
        var html5QrcodeScanner = new Html5Qeader(
            "qr-reader", { fps: 10, qrbox: 250 });
        html5QrcodeScanner.render(onScanSuccess);

        // Função para enviar os dados para a planilha
        function submitForm() {
            const form = new FormData();
            form.append('codigo', document.getElementById('codigo').value);
            form.append('tipo', document.getElementById('tipo').value);
            form.append('quantidade', document.getElementById('quantidade').value);
            form.append('nome', document.getElementById('nome').value);

            document.querySelector('button').disabled = true;
            document.querySelector('button').textContent = 'Enviando...';
            
            fetch(SCRIPT_URL, { method: 'POST', body: form})
                .then(response => response.text())
                .then(data => {
                    document.getElementById('success-message').textContent = "Registrado com sucesso! Recarregue a página para uma nova leitura.";
                    document.getElementById('success-message').style.display = 'block';
                    formContainer.style.display = 'none';
                })
                .catch(error => {
                    document.getElementById('error-message').textContent = 'Erro ao registrar. Tente novamente.';
                    document.getElementById('error-message').style.display = 'block';
                    document.querySelector('button').disabled = false;
                    document.querySelector('button').textContent = 'Registrar Movimentação';
                    console.error('Error:', error);
                });
        }
    </script>
</body>
</html>
