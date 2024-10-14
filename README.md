<Controle de estoque e vendas>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Estoque</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
        #overlay {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
        }
        .modal {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            padding: 20px;
            border-radius: 5px;
            min-width: 300px;
        }
    </style>
</head>
<body>

<h1>Centro de Distribuição</h1>
<div id="loginScreen" class="modal">
    <h2>Login</h2>
    <input type="text" id="username" placeholder="Usuário" required>
    <input type="password" id="password" placeholder="Senha" required>
    <button id="loginBtn">Entrar</button>
    <p id="loginError" style="color: red; display: none;">Usuário ou senha inválidos.</p>
</div>

<div id="mainContent" style="display:none;">
    <h2>Informações do Estabelecimento</h2>
    <form id="establishmentForm">
        <input type="text" id="establishmentName" placeholder="Nome do Estabelecimento" required>
        <input type="text" id="establishmentCNPJ" placeholder="CNPJ" value="12.345.678/0001-90" required>
        <input type="text" id="establishmentAddress" placeholder="Endereço" value="Rua Exemplo, 123 - Cidade - Estado" required>
        <input type="email" id="establishmentEmail" placeholder="Email" value="contato@exemplo.com" required>
        <input type="text" id="establishmentPhone" placeholder="Telefone" value="(11) 1234-5678" required>
        <button type="submit">Salvar Informações</button>
    </form>

    <h2>Cadastro de Produtos</h2>
    <form id="productForm">
        <input type="text" id="productName" placeholder="Nome do Produto" required>
        <input type="number" id="productQuantity" placeholder="Quantidade" required>
        <input type="text" id="productBrand" placeholder="Marca" required>
        <input type="text" id="productLocation" placeholder="Localização" required>
        <input type="number" step="0.01" id="productPrice" placeholder="Preço" required>
        <input type="text" id="productBarcode" placeholder="Código de Barras" required>
        <button type="submit">Cadastrar Produto</button>
    </form>

    <h2>Lista de Produtos</h2>
    <table id="productTable">
        <thead>
            <tr>
                <th>Nome</th>
                <th>Quantidade</th>
                <th>Marca</th>
                <th>Localização</th>
                <th>Preço</th>
                <th>Código de Barras</th>
                <th>Ações</th>
            </tr>
        </thead>
        <tbody>
            <!-- Produtos cadastrados aparecerão aqui -->
        </tbody>
    </table>

    <h2>Vendas</h2>
    <button id="startSaleBtn">Iniciar Venda</button>
    <button id="viewSalesHistoryBtn">Histórico de Vendas</button>

    <div id="overlay">
        <div class="modal" id="saleScreen" style="display:none;">
            <h2>Venda de Produto</h2>
            <input type="text" id="saleIdentifier" placeholder="Código de Barras ou Nome do Produto" required>
            <input type="number" id="saleQuantity" placeholder="Quantidade" required>
            <button id="addProductToSaleBtn">Adicionar Produto</button>
            <button id="completeSaleBtn" style="display:none;">Finalizar Venda</button>
            <button id="cancelSaleBtn">Cancelar Venda</button>
            <div id="addedProductsList" style="margin-top: 10px;"></div>
            <div id="totalSaleValue" style="margin-top: 10px;"><strong>Valor Total: R$ <span id="totalValue">0.00</span></strong></div>
        </div>

        <div class="modal" id="receiptScreen" style="display:none;">
            <h2>Comprovante de Venda</h2>
            <div id="receiptProducts"></div>
            <p><strong>Valor Total:</strong> R$ <span id="receiptTotalValue"></span></p>
            <p><strong>Valor Pago:</strong> R$ <span id="receiptPayment"></span></p>
            <p><strong>Troco:</strong> R$ <span id="receiptChange"></span></p>
            <p><strong>Data:</strong> <span id="receiptDate"></span></p>
            <p><strong>Nome do Estabelecimento:</strong> <span id="receiptEstablishmentName"></span></p>
            <p><strong>CNPJ:</strong> <span id="receiptEstablishmentCNPJ"></span></p>
            <p><strong>Endereço:</strong> <span id="receiptEstablishmentAddress"></span></p>
            <p><strong>Email:</strong> <span id="receiptEstablishmentEmail"></span></p>
            <p><strong>Telefone:</strong> <span id="receiptEstablishmentPhone"></span></p>
            <button id="printReceiptBtn">Imprimir Comprovante</button>
            <button id="closeReceiptBtn">Fechar</button>
        </div>

        <div class="modal" id="salesHistoryScreen" style="display:none;">
            <h2>Histórico de Vendas</h2>
            <table id="salesHistoryTable">
                <thead>
                    <tr>
                        <th>Produto</th>
                        <th>Quantidade</th>
                        <th>Valor Pago</th>
                        <th>Troco</th>
                        <th>Data</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- Histórico de vendas aparecerá aqui -->
                </tbody>
            </table>
            <button id="closeSalesHistoryBtn">Fechar</button>
        </div>
    </div>
</div>

<script>
    const salesHistory = [];
    const products = [];
    let totalSale = 0;
    let addedProducts = [];

    document.getElementById('loginBtn').addEventListener('click', function() {
        const username = document.getElementById('username').value;
        const password = document.getElementById('password').value;

        if (username === "loja01" && password === "010203") {
            document.getElementById('loginScreen').style.display = 'none';
            document.getElementById('mainContent').style.display = 'block';
        } else {
            document.getElementById('loginError').style.display = 'block';
        }
    });

    document.getElementById('establishmentForm').addEventListener('submit', function(event) {
        event.preventDefault(); // Previne o envio do formulário
        alert('Informações do estabelecimento salvas com sucesso!');
    });

    document.getElementById('productForm').addEventListener('submit', function(event) {
        event.preventDefault();

        const productName = document.getElementById('productName').value;
        const productQuantity = parseInt(document.getElementById('productQuantity').value);
        const productBrand = document.getElementById('productBrand').value;
        const productLocation = document.getElementById('productLocation').value;
        const productPrice = parseFloat(document.getElementById('productPrice').value);
        const productBarcode = document.getElementById('productBarcode').value;

        products.push({
            name: productName,
            quantity: productQuantity,
            brand: productBrand,
            location: productLocation,
            price: productPrice,
            barcode: productBarcode
        });

        saveProducts(); // Salvar produtos após adicionar
        updateProductTable();
        this.reset();
    });

    function updateProductTable() {
        const tbody = document.querySelector('#productTable tbody');
        tbody.innerHTML = '';
        products.forEach((product, index) => {
            tbody.innerHTML += `
                <tr>
                    <td>${product.name}</td>
                    <td>${product.quantity}</td>
                    <td>${product.brand}</td>
                    <td>${product.location}</td>
                    <td>R$ ${product.price.toFixed(2)}</td>
                    <td>${product.barcode}</td>
                    <td>
                        <button onclick="removeProduct(${index})">Remover</button>
                    </td>
                </tr>
            `;
        });
    }

    function removeProduct(index) {
        products.splice(index, 1);
        updateProductTable();
    }

    function saveProducts() {
        localStorage.setItem('products', JSON.stringify(products));
    }

    function loadProducts() {
        const savedProducts = JSON.parse(localStorage.getItem('products'));
        if (savedProducts) {
            products.push(...savedProducts);
            updateProductTable();
        }
    }

    document.getElementById('startSaleBtn').addEventListener('click', function() {
        totalSale = 0;
        addedProducts = [];
        document.getElementById('addedProductsList').innerHTML = '';
        document.getElementById('totalValue').textContent = '0.00';
        document.getElementById('saleScreen').style.display = 'block';
        document.getElementById('overlay').style.display = 'block';
    });

    document.getElementById('addProductToSaleBtn').addEventListener('click', function() {
        const identifier = document.getElementById('saleIdentifier').value;
        const quantity = parseInt(document.getElementById('saleQuantity').value);
        const product = products.find(p => p.name === identifier || p.barcode === identifier);

        if (product && product.quantity >= quantity) {
            product.quantity -= quantity; // Atualiza a quantidade do produto no estoque
            const productTotal = product.price * quantity;

            totalSale += productTotal;
            addedProducts.push({ ...product, quantity });

            document.getElementById('addedProductsList').innerHTML += `<div>${product.name} - Qtd: ${quantity} - R$ ${productTotal.toFixed(2)}</div>`;
            document.getElementById('totalValue').textContent = totalSale.toFixed(2);

            updateProductTable(); // Atualiza a tabela de produtos após a venda

            if (totalSale > 0) {
                document.getElementById('completeSaleBtn').style.display = 'inline-block';
            }
        } else {
            alert('Produto não encontrado ou quantidade insuficiente.');
        }

        document.getElementById('saleIdentifier').value = '';
        document.getElementById('saleQuantity').value = '';
    });

    document.getElementById('completeSaleBtn').addEventListener('click', function() {
        const payment = parseFloat(prompt('Valor pago:'));
        if (payment >= totalSale) {
            const change = payment - totalSale;

            salesHistory.push({
                products: addedProducts,
                totalValue: totalSale,
                change: change,
                date: new Date().toLocaleString()
            });

            saveSalesHistory(); // Salvar histórico de vendas após finalizar a venda
            displayReceipt(payment, change);
        } else {
            alert('Valor pago insuficiente.');
        }
    });

    function displayReceipt(payment, change) {
        const receiptProducts = document.getElementById('receiptProducts');
        receiptProducts.innerHTML = ''; // Limpa o recibo anterior
        addedProducts.forEach(item => {
            receiptProducts.innerHTML += `<div>${item.name} - Qtd: ${item.quantity} - R$ ${(item.price * item.quantity).toFixed(2)}</div>`;
        });

        document.getElementById('receiptTotalValue').textContent = totalSale.toFixed(2);
        document.getElementById('receiptPayment').textContent = payment.toFixed(2);
        document.getElementById('receiptChange').textContent = change.toFixed(2);
        document.getElementById('receiptDate').textContent = new Date().toLocaleString();
        document.getElementById('receiptEstablishmentName').textContent = document.getElementById('establishmentName').value;
        document.getElementById('receiptEstablishmentCNPJ').textContent = document.getElementById('establishmentCNPJ').value;
        document.getElementById('receiptEstablishmentAddress').textContent = document.getElementById('establishmentAddress').value;
        document.getElementById('receiptEstablishmentEmail').textContent = document.getElementById('establishmentEmail').value;
        document.getElementById('receiptEstablishmentPhone').textContent = document.getElementById('establishmentPhone').value;

        // Exibir recibo
        document.getElementById('saleScreen').style.display = 'none';
        document.getElementById('receiptScreen').style.display = 'block';
        document.getElementById('overlay').style.display = 'block'; // Certifica-se que o overlay também está visível
    }

    document.getElementById('closeReceiptBtn').addEventListener('click', function() {
        document.getElementById('receiptScreen').style.display = 'none';
        document.getElementById('overlay').style.display = 'none'; // Oculta o overlay
    });

    document.getElementById('printReceiptBtn').addEventListener('click', function() {
        window.print(); // Imprime o recibo
    });

    document.getElementById('viewSalesHistoryBtn').addEventListener('click', function() {
        loadSalesHistory();
        document.getElementById('salesHistoryScreen').style.display = 'block';
        document.getElementById('overlay').style.display = 'block'; // Mostra o overlay
    });

    document.getElementById('closeSalesHistoryBtn').addEventListener('click', function() {
        document.getElementById('salesHistoryScreen').style.display = 'none';
        document.getElementById('overlay').style.display = 'none'; // Oculta o overlay
    });

    function saveSalesHistory() {
        localStorage.setItem('salesHistory', JSON.stringify(salesHistory));
    }

    function loadSalesHistory() {
        const savedSales = JSON.parse(localStorage.getItem('salesHistory'));
        const tbody = document.querySelector('#salesHistoryTable tbody');
        tbody.innerHTML = '';
        if (savedSales) {
            savedSales.forEach(sale => {
                sale.products.forEach(item => {
                    tbody.innerHTML += `
                        <tr>
                            <td>${item.name}</td>
                            <td>${item.quantity}</td>
                            <td>R$ ${sale.totalValue.toFixed(2)}</td>
                            <td>R$ ${sale.change.toFixed(2)}</td>
                            <td>${sale.date}</td>
                        </tr>
                    `;
                });
            });
        }
    }

    loadProducts(); // Carrega os produtos ao iniciar
</script>
</body>
</html>
