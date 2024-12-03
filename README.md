<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplikasi Transaksi</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.3/css/all.min.css"></link>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
</head>
<body class="bg-gray-100 font-roboto">
    <div class="container mx-auto p-4">
        <h1 class="text-3xl font-bold mb-6 text-center text-blue-600">Aplikasi Transaksi</h1>
        
        <div class="bg-white p-6 rounded-lg shadow-lg mb-6">
            <h2 class="text-2xl font-semibold mb-4 text-blue-500">Tambah Transaksi</h2>
            <form id="transaction-form" class="space-y-4">
                <div>
                    <label for="description" class="block text-sm font-medium text-gray-700">Deskripsi</label>
                    <input type="text" id="description" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2 focus:ring-blue-500 focus:border-blue-500" required>
                </div>
                <div>
                    <label for="amount" class="block text-sm font-medium text-gray-700">Jumlah</label>
                    <input type="number" id="amount" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2 focus:ring-blue-500 focus:border-blue-500" required>
                </div>
                <div>
                    <label for="type" class="block text-sm font-medium text-gray-700">Tipe</label>
                    <select id="type" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2 focus:ring-blue-500 focus:border-blue-500" required>
                        <option value="income">Pemasukan</option>
                        <option value="expense">Pengeluaran</option>
                    </select>
                </div>
                <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded-md shadow-sm hover:bg-blue-600">Tambah Transaksi</button>
            </form>
        </div>

        <div class="bg-white p-6 rounded-lg shadow-lg mb-6">
            <h2 class="text-2xl font-semibold mb-4 text-blue-500">Saldo</h2>
            <p id="balance" class="text-3xl font-bold text-green-500">Rp0,00</p>
        </div>

        <div class="bg-white p-6 rounded-lg shadow-lg mb-6">
            <h2 class="text-2xl font-semibold mb-4 text-blue-500">Transaksi</h2>
            <div id="transactions" class="space-y-4">
                <!-- Transaksi akan ditambahkan secara dinamis di sini -->
            </div>
        </div>

        <div class="bg-white p-6 rounded-lg shadow-lg mb-6">
            <h2 class="text-2xl font-semibold mb-4 text-blue-500">Transaksi Terkelompok</h2>
            <div class="space-y-4">
                <div>
                    <h3 class="text-xl font-semibold text-blue-400">Harian</h3>
                    <div id="daily-transactions" class="space-y-2">
                        <!-- Transaksi harian akan ditambahkan secara dinamis di sini -->
                    </div>
                </div>
                <div>
                    <h3 class="text-xl font-semibold text-blue-400">Mingguan</h3>
                    <div id="weekly-transactions" class="space-y-2">
                        <!-- Transaksi mingguan akan ditambahkan secara dinamis di sini -->
                    </div>
                </div>
                <div>
                    <h3 class="text-xl font-semibold text-blue-400">Bulanan</h3>
                    <div id="monthly-transactions" class="space-y-2">
                        <!-- Transaksi bulanan akan ditambahkan secara dinamis di sini -->
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        document.getElementById('transaction-form').addEventListener('submit', function(event) {
            event.preventDefault();
            
            const description = document.getElementById('description').value;
            const amount = parseFloat(document.getElementById('amount').value);
            const type = document.getElementById('type').value;
            
            const transaction = {
                description,
                amount,
                type,
                date: new Date()
            };
            
            addTransaction(transaction);
            updateBalance();
            groupTransactions();
        });

        let transactions = [];

        function addTransaction(transaction) {
            transactions.push(transaction);
            const transactionElement = document.createElement('div');
            transactionElement.classList.add('flex', 'justify-between', 'items-center', 'p-2', 'border', 'border-gray-300', 'rounded-md', 'bg-gray-50');
            transactionElement.innerHTML = `
                <span>${transaction.description}</span>
                <span class="${transaction.type === 'income' ? 'text-green-500' : 'text-red-500'}">${transaction.type === 'income' ? '+' : '-'}Rp${transaction.amount.toLocaleString('id-ID', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}</span>
                <span class="text-gray-500 text-sm">${transaction.date.toLocaleString()}</span>
            `;
            document.getElementById('transactions').appendChild(transactionElement);
        }

        function updateBalance() {
            const balance = transactions.reduce((acc, transaction) => {
                return transaction.type === 'income' ? acc + transaction.amount : acc - transaction.amount;
            }, 0);
            document.getElementById('balance').textContent = `Rp${balance.toLocaleString('id-ID', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
        }

        function groupTransactions() {
            const dailyTransactions = {};
            const weeklyTransactions = {};
            const monthlyTransactions = {};

            transactions.forEach(transaction => {
                const date = transaction.date.toISOString().split('T')[0];
                const week = getWeekNumber(transaction.date);
                const month = transaction.date.getMonth() + 1;

                if (!dailyTransactions[date]) dailyTransactions[date] = [];
                if (!weeklyTransactions[week]) weeklyTransactions[week] = [];
                if (!monthlyTransactions[month]) monthlyTransactions[month] = [];

                dailyTransactions[date].push(transaction);
                weeklyTransactions[week].push(transaction);
                monthlyTransactions[month].push(transaction);
            });

            displayGroupedTransactions(dailyTransactions, 'daily-transactions');
            displayGroupedTransactions(weeklyTransactions, 'weekly-transactions');
            displayGroupedTransactions(monthlyTransactions, 'monthly-transactions');
        }

        function displayGroupedTransactions(groupedTransactions, elementId) {
            const container = document.getElementById(elementId);
            container.innerHTML = '';
            for (const [key, transactions] of Object.entries(groupedTransactions)) {
                const groupElement = document.createElement('div');
                groupElement.classList.add('p-2', 'border', 'border-gray-300', 'rounded-md', 'bg-gray-50');
                groupElement.innerHTML = `
                    <h4 class="font-semibold text-blue-400">${key}</h4>
                    <div class="space-y-1">
                        ${transactions.map(transaction => `
                            <div class="flex justify-between items-center">
                                <span>${transaction.description}</span>
                                <span class="${transaction.type === 'income' ? 'text-green-500' : 'text-red-500'}">${transaction.type === 'income' ? '+' : '-'}Rp${transaction.amount.toLocaleString('id-ID', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}</span>
                                <span class="text-gray-500 text-sm">${transaction.date.toLocaleString()}</span>
                            </div>
                        `).join('')}
                    </div>
                `;
                container.appendChild(groupElement);
            }
        }

        function getWeekNumber(date) {
            const firstDayOfYear = new Date(date.getFullYear(), 0, 1);
            const pastDaysOfYear = (date - firstDayOfYear) / 86400000;
            return Math.ceil((pastDaysOfYear + firstDayOfYear.getDay() + 1) / 7);
        }
    </script>
</body>
</html>
