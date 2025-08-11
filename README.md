<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RAPR Entregas®</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #111827; /* bg-gray-900 */
            color: #f3f4f6; /* text-gray-200 */
        }
        .kpi-card {
            background-color: #1f2937; /* bg-gray-800 */
            border-radius: 0.75rem; /* rounded-xl */
            padding: 1.5rem; /* p-6 */
            border: 1px solid #374151; /* border-gray-700 */
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .kpi-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        .chart-container {
            background-color: #1f2937; /* bg-gray-800 */
            border-radius: 0.75rem; /* rounded-xl */
            padding: 1.5rem; /* p-6 */
            border: 1px solid #374151; /* border-gray-700 */
        }
        .modal-backdrop {
            background-color: rgba(0, 0, 0, 0.7);
        }
        .modal-content {
            background-color: #1f2937; /* bg-gray-800 */
        }
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #1f2937;
        }
        ::-webkit-scrollbar-thumb {
            background: #4b5563;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #6b7280;
        }
    </style>
</head>
<body class="p-4 sm:p-6 lg:p-8">

    <div class="max-w-7xl mx-auto">
        <!-- Cabeçalho -->
        <header class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-8 gap-4">
            <div class="flex items-center gap-4">
                <img id="dashboardLogo" src="" alt="Logo RAPR Entregas" class="h-20 object-contain bg-white p-1 rounded-md">
                <div>
                    <h1 class="text-3xl font-bold text-white">RAPR Entregas®</h1>
                    <p class="text-gray-400 mt-1">Análise de performance de entregas</p>
                </div>
            </div>
            <div class="flex flex-wrap gap-2 justify-start sm:justify-end">
                 <button id="exportEntregasBtn" class="bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded-lg shadow-md transition-transform transform hover:scale-105">Relatório de Entregas</button>
                 <button id="exportCustosBtn" class="bg-yellow-600 hover:bg-yellow-700 text-white font-bold py-2 px-4 rounded-lg shadow-md transition-transform transform hover:scale-105">Relatório de Custos</button>
                 <button id="addEntryBtn" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg shadow-md transition-transform transform hover:scale-105">+ Adicionar Corrida</button>
            </div>
        </header>

        <!-- KPIs - Indicadores Chave de Performance -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-6 mb-8">
            <div class="kpi-card">
                <h3 class="text-gray-400 text-sm font-medium">Receita Bruta Total</h3>
                <p id="kpi-receita-bruta" class="text-3xl font-bold text-green-400 mt-2">R$ 0,00</p>
            </div>
            <div class="kpi-card">
                <h3 class="text-gray-400 text-sm font-medium">Custos Totais</h3>
                <p id="kpi-custos-totais" class="text-3xl font-bold text-red-400 mt-2">R$ 0,00</p>
            </div>
            <div class="kpi-card">
                <h3 class="text-gray-400 text-sm font-medium">Lucro Líquido Total</h3>
                <p id="kpi-lucro-liquido" class="text-3xl font-bold text-cyan-400 mt-2">R$ 0,00</p>
            </div>
            <div class="kpi-card">
                <h3 class="text-gray-400 text-sm font-medium">Total de Entregas</h3>
                <p id="kpi-total-entregas" class="text-3xl font-bold text-white mt-2">0</p>
            </div>
            <div class="kpi-card">
                <h3 class="text-gray-400 text-sm font-medium">Lucro Médio / Corrida</h3>
                <p id="kpi-lucro-medio" class="text-3xl font-bold text-yellow-400 mt-2">R$ 0,00</p>
            </div>
        </div>
        
        <!-- Gráficos -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
            <div class="chart-container">
                <h3 class="text-lg font-semibold text-white mb-4">Lucro por Plataforma</h3>
                <canvas id="lucroPorPlataformaChart" class="max-h-80 mx-auto"></canvas>
            </div>
            <div class="chart-container">
                <h3 class="text-lg font-semibold text-white mb-4">Análise de Custos</h3>
                <canvas id="analiseCustosChart" class="max-h-80 mx-auto"></canvas>
            </div>
        </div>

        <!-- Tabela de Lançamentos -->
        <div class="bg-gray-800 rounded-xl p-6 border border-gray-700">
            <h3 class="text-lg font-semibold text-white mb-4">Histórico de Corridas</h3>
            <div class="overflow-x-auto">
                <table class="w-full text-sm text-left text-gray-400">
                    <thead class="text-xs text-gray-300 uppercase bg-gray-700">
                        <tr>
                            <th scope="col" class="px-6 py-3">Data</th>
                            <th scope="col" class="px-6 py-3">Plataforma</th>
                            <th scope="col" class="px-6 py-3">Receita Bruta</th>
                            <th scope="col" class="px-6 py-3">Custos</th>
                            <th scope="col" class="px-6 py-3">Lucro Líquido</th>
                            <th scope="col" class="px-6 py-3">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="entriesTableBody">
                        <!-- Linhas da tabela serão inseridas aqui via JS -->
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <!-- Modal para Adicionar/Editar Lançamento -->
    <div id="entryModal" class="fixed inset-0 z-50 items-center justify-center hidden p-4 modal-backdrop">
        <div class="modal-content w-full max-w-lg mx-auto rounded-xl shadow-lg p-6 border border-gray-700">
            <h2 id="modalTitle" class="text-2xl font-bold text-white mb-6">Adicionar Nova Corrida</h2>
            <form id="entryForm">
                <input type="hidden" id="entryId">
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <div>
                        <label for="date" class="block text-sm font-medium text-gray-300 mb-1">Data</label>
                        <input type="date" id="date" class="w-full bg-gray-700 border border-gray-600 text-white rounded-lg p-2.5 focus:ring-indigo-500 focus:border-indigo-500" required>
                    </div>
                    <div>
                        <label for="platform" class="block text-sm font-medium text-gray-300 mb-1">Plataforma</label>
                        <select id="platform" class="w-full bg-gray-700 border border-gray-600 text-white rounded-lg p-2.5 focus:ring-indigo-500 focus:border-indigo-500">
                            <option>Lalamove</option>
                            <option>99 Entregas</option>
                            <option>Uber Flash</option>
                            <option>Outra</option>
                        </select>
                    </div>
                    <div>
                        <label for="valorEntrega" class="block text-sm font-medium text-gray-300 mb-1">Valor da Entrega (R$)</label>
                        <input type="number" id="valorEntrega" step="0.01" class="w-full bg-gray-700 border border-gray-600 text-white rounded-lg p-2.5" placeholder="55.00" required>
                    </div>
                    <div>
                        <label for="taxaPlataforma" class="block text-sm font-medium text-gray-300 mb-1">Taxa da Plataforma (%)</label>
                        <input type="number" id="taxaPlataforma" step="0.01" class="w-full bg-gray-700 border border-gray-600 text-white rounded-lg p-2.5" placeholder="20">
                    </div>
                    <div>
                        <label for="combustivel" class="block text-sm font-medium text-gray-300 mb-1">Custo Combustível (R$)</label>
                        <input type="number" id="combustivel" step="0.01" class="w-full bg-gray-700 border border-gray-600 text-white rounded-lg p-2.5" placeholder="10.00">
                    </div>
                    <div>
                        <label for="pedagio" class="block text-sm font-medium text-gray-300 mb-1">Pedágio (R$)</label>
                        <input type="number" id="pedagio" step="0.01" class="w-full bg-gray-700 border border-gray-600 text-white rounded-lg p-2.5" placeholder="6.50">
                    </div>
                    <div class="sm:col-span-2">
                        <label for="outrosCustos" class="block text-sm font-medium text-gray-300 mb-1">Outros Custos (R$)</label>
                        <input type="number" id="outrosCustos" step="0.01" class="w-full bg-gray-700 border border-gray-600 text-white rounded-lg p-2.5" placeholder="5.00">
                    </div>
                </div>
                <div class="flex justify-end gap-4 mt-6">
                    <button type="button" id="cancelBtn" class="bg-gray-600 hover:bg-gray-500 text-white font-bold py-2 px-4 rounded-lg">Cancelar</button>
                    <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg">Salvar</button>
                </div>
            </form>
        </div>
    </div>


    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const logoBase64 = 'data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/2wBDAAMCAgICAgMCAgIDAwMDBAYEBAQEBAgGBgUGCQgKCgkICQkKDA8MCgsOCwkJDRENDg8QEBEQCgwSExIQEw8QEBD/2wBDAQMDAwQDBAgEBAgQCwkLEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBD/wAARCAGQAZADASIAAhEBAxEB/8QAHwAAAQUBAQEBAQEAAAAAAAAAAAECAwQFBgcICQoL/8QAtRAAAgEDAwIEAwUFBAQAAAF9AQIDAAQRBRIhMUEGE1FhByJxFDKBkaEII0KxwRVS0fAkM2JyggkKFhcYGRolJicoKSo0NTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqDhIWGh4iJipKTlJWWl5iZmqKjpKWmp6ipqrKztLW2t7i5usLDxMXGx8jJytLT1NXW19jZ2uHi4+Tl5ufo6erx8vP09fb3+Pn6/8QAHwEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoL/8QAtREAAgECBAQDBAcFBAQAAQJ3AAECAxEEBSExBhJBUQdhcRMiMoEIFEKRobHBCSMzUvAVYnLRChYkNOEl8RcYGRomJygpKjU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6goOEhYaHiImKkpOUlZaXmJmaoqOkpaanqKmqsrO0tba3uLm6wsPExcbHyMnK0tPU1dbX2Nna4uPk5ebn6Onq8vP09fb3+Pn6/9oADAMBAAIRAxEAPwD9/KKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigA-';

            // Elementos da UI
            const dashboardLogo = document.getElementById('dashboardLogo');
            const addEntryBtn = document.getElementById('addEntryBtn');
            const exportEntregasBtn = document.getElementById('exportEntregasBtn');
            const exportCustosBtn = document.getElementById('exportCustosBtn');
            const entryModal = document.getElementById('entryModal');
            const modalTitle = document.getElementById('modalTitle');
            const cancelBtn = document.getElementById('cancelBtn');
            const entryForm = document.getElementById('entryForm');
            const entriesTableBody = document.getElementById('entriesTableBody');

            // CORREÇÃO: Atribui a imagem Base64 ao logo do painel
            dashboardLogo.src = logoBase64;

            // Variáveis de estado
            let entries = [];
            let lucroPorPlataformaChart, analiseCustosChart;

            // Funções de formatação
            const formatCurrency = (value) => new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value);
            
            // Funções do Modal
            const showModal = () => {
                entryModal.classList.remove('hidden');
                entryModal.classList.add('flex');
            };
            const hideModal = () => {
                entryModal.classList.add('hidden');
                entryModal.classList.remove('flex');
                entryForm.reset();
                document.getElementById('entryId').value = '';
            };

            // Lógica de dados
            const loadEntries = () => {
                const storedEntries = localStorage.getItem('deliveryEntries');
                if (storedEntries) {
                    entries = JSON.parse(storedEntries);
                } else {
                    entries = [
                        { id: 1, date: '2025-07-15', platform: 'Lalamove', valorEntrega: 65.50, taxaPlataforma: 15.5, combustivel: 12.00, pedagio: 0, outrosCustos: 2.50 },
                        { id: 2, date: '2025-07-18', platform: '99 Entregas', valorEntrega: 45.00, taxaPlataforma: 16.67, combustivel: 8.00, pedagio: 0, outrosCustos: 0 },
                        { id: 3, date: '2025-08-01', platform: 'Lalamove', valorEntrega: 120.00, taxaPlataforma: 18.33, combustivel: 25.00, pedagio: 6.50, outrosCustos: 10.00 },
                        { id: 4, date: '2025-08-03', platform: 'Uber Flash', valorEntrega: 35.80, taxaPlataforma: 16.76, combustivel: 5.50, pedagio: 0, outrosCustos: 0 },
                    ];
                    saveEntries();
                }
            };

            const saveEntries = () => {
                localStorage.setItem('deliveryEntries', JSON.stringify(entries));
            };

            const addOrUpdateEntry = (e) => {
                e.preventDefault();
                const entryData = {
                    id: document.getElementById('entryId').value ? Number(document.getElementById('entryId').value) : Date.now(),
                    date: document.getElementById('date').value,
                    platform: document.getElementById('platform').value,
                    valorEntrega: parseFloat(document.getElementById('valorEntrega').value) || 0,
                    taxaPlataforma: parseFloat(document.getElementById('taxaPlataforma').value) || 0,
                    combustivel: parseFloat(document.getElementById('combustivel').value) || 0,
                    pedagio: parseFloat(document.getElementById('pedagio').value) || 0,
                    outrosCustos: parseFloat(document.getElementById('outrosCustos').value) || 0,
                };

                const existingIndex = entries.findIndex(entry => entry.id === entryData.id);
                if (existingIndex > -1) {
                    entries[existingIndex] = entryData;
                } else {
                    entries.push(entryData);
                }
                
                saveEntries();
                renderAll();
                hideModal();
            };

            const handleEdit = (id) => {
                const entry = entries.find(e => e.id === id);
                if (!entry) return;

                modalTitle.textContent = 'Editar Corrida';
                document.getElementById('entryId').value = entry.id;
                document.getElementById('date').value = entry.date;
                document.getElementById('platform').value = entry.platform;
                document.getElementById('valorEntrega').value = entry.valorEntrega;
                document.getElementById('taxaPlataforma').value = entry.taxaPlataforma;
                document.getElementById('combustivel').value = entry.combustivel;
                document.getElementById('pedagio').value = entry.pedagio;
                document.getElementById('outrosCustos').value = entry.outrosCustos;
                
                showModal();
            };

            const deleteEntry = (id) => {
                if (confirm('Tem certeza que deseja excluir esta corrida?')) {
                    entries = entries.filter(entry => entry.id !== id);
                    saveEntries();
                    renderAll();
                }
            };
            
            // Funções de renderização
            const renderTable = () => {
                entriesTableBody.innerHTML = '';
                if (entries.length === 0) {
                    entriesTableBody.innerHTML = `<tr><td colspan="6" class="text-center px-6 py-4">Nenhuma corrida registrada.</td></tr>`;
                    return;
                }
                
                const sortedEntries = [...entries].sort((a, b) => new Date(b.date) - new Date(a.date));

                sortedEntries.forEach(entry => {
                    const taxaValor = (entry.valorEntrega * entry.taxaPlataforma) / 100;
                    const custosTotais = taxaValor + entry.combustivel + entry.pedagio + entry.outrosCustos;
                    const lucroLiquido = entry.valorEntrega - custosTotais;

                    const row = document.createElement('tr');
                    row.className = 'bg-gray-800 border-b border-gray-700 hover:bg-gray-600';
                    
                    row.innerHTML = `
                        <td class="px-6 py-4">${new Date(entry.date + 'T00:00:00').toLocaleDateString('pt-BR')}</td>
                        <td class="px-6 py-4 font-medium text-white">${entry.platform}</td>
                        <td class="px-6 py-4 text-green-400">${formatCurrency(entry.valorEntrega)}</td>
                        <td class="px-6 py-4 text-red-400">${formatCurrency(custosTotais)}</td>
                        <td class="px-6 py-4 text-cyan-400 font-bold">${formatCurrency(lucroLiquido)}</td>
                        <td class="px-6 py-4">
                            <button class="font-medium text-indigo-400 hover:text-indigo-600 mr-4 edit-btn" data-id="${entry.id}">Editar</button>
                            <button class="font-medium text-red-500 hover:text-red-700 delete-btn" data-id="${entry.id}">Excluir</button>
                        </td>
                    `;
                    entriesTableBody.appendChild(row);
                });
            };

            const updateKPIs = () => {
                const totalReceitaBruta = entries.reduce((sum, entry) => sum + entry.valorEntrega, 0);
                const totalCustos = entries.reduce((sum, entry) => {
                    const taxaValor = (entry.valorEntrega * entry.taxaPlataforma) / 100;
                    return sum + taxaValor + entry.combustivel + entry.pedagio + entry.outrosCustos;
                }, 0);
                const totalLucroLiquido = totalReceitaBruta - totalCustos;
                const totalEntregas = entries.length;
                const lucroMedio = totalEntregas > 0 ? totalLucroLiquido / totalEntregas : 0;

                document.getElementById('kpi-receita-bruta').textContent = formatCurrency(totalReceitaBruta);
                document.getElementById('kpi-custos-totais').textContent = formatCurrency(totalCustos);
                document.getElementById('kpi-lucro-liquido').textContent = formatCurrency(totalLucroLiquido);
                document.getElementById('kpi-total-entregas').textContent = totalEntregas;
                document.getElementById('kpi-lucro-medio').textContent = formatCurrency(lucroMedio);
            };

            const updateCharts = () => {
                const lucroPorPlataformaData = {};
                entries.forEach(entry => {
                    const taxaValor = (entry.valorEntrega * entry.taxaPlataforma) / 100;
                    const lucro = entry.valorEntrega - (taxaValor + entry.combustivel + entry.pedagio + entry.outrosCustos);
                    if (!lucroPorPlataformaData[entry.platform]) {
                        lucroPorPlataformaData[entry.platform] = 0;
                    }
                    lucroPorPlataformaData[entry.platform] += lucro;
                });
                const lucroLabels = Object.keys(lucroPorPlataformaData);
                const lucroValues = Object.values(lucroPorPlataformaData);

                if (lucroPorPlataformaChart) lucroPorPlataformaChart.destroy();
                lucroPorPlataformaChart = new Chart(document.getElementById('lucroPorPlataformaChart'), {
                    type: 'pie',
                    data: {
                        labels: lucroLabels,
                        datasets: [{
                            data: lucroValues,
                            backgroundColor: ['#34D399', '#F87171', '#60A5FA', '#FBBF24', '#A78BFA', '#F472B6'],
                            borderColor: '#1f2937',
                        }]
                    },
                    options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'top', labels: { color: '#d1d5db' } } } }
                });

                const custosData = {
                    'Taxa Plataforma': entries.reduce((sum, e) => sum + ((e.valorEntrega * e.taxaPlataforma) / 100), 0),
                    'Combustível': entries.reduce((sum, e) => sum + e.combustivel, 0),
                    'Pedágio': entries.reduce((sum, e) => sum + e.pedagio, 0),
                    'Outros Custos': entries.reduce((sum, e) => sum + e.outrosCustos, 0),
                };
                const custosLabels = Object.keys(custosData);
                const custosValues = Object.values(custosData);

                if (analiseCustosChart) analiseCustosChart.destroy();
                analiseCustosChart = new Chart(document.getElementById('analiseCustosChart'), {
                    type: 'pie',
                    data: {
                        labels: custosLabels,
                        datasets: [{
                            data: custosValues,
                            backgroundColor: ['#FB923C', '#F472B6', '#A78BFA', '#A3A3A3'],
                            borderColor: '#1f2937',
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: { 
                            legend: { position: 'top', labels: { color: '#d1d5db' } },
                            datalabels: {
                                formatter: (value, ctx) => {
                                    if (value > 0) { return formatCurrency(value); }
                                    return null;
                                },
                                color: '#FFFFFF',
                                font: { weight: 'bold' }
                            }
                        }
                    },
                    plugins: [ChartDataLabels]
                });
            };
            
            // Funções de Exportação
            const getReportHTML = (title, tableContent) => {
                return `
                    <!DOCTYPE html>
                    <html lang="pt-BR">
                    <head>
                        <meta charset="UTF-8">
                        <title>${title}</title>
                        <style>
                            body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 2rem; }
                            table { width: 100%; border-collapse: collapse; margin-top: 1rem; }
                            th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
                            th { background-color: #f2f2f2; }
                            .report-header { display: flex; align-items: center; gap: 20px; border-bottom: 2px solid #ccc; padding-bottom: 1rem; }
                            .report-header img { height: 80px; object-fit: contain; }
                            .report-header h1 { margin: 0; }
                            .month-header { background-color: #e0f2fe; color: #0c4a6e; font-size: 1.2em; font-weight: bold; }
                            .month-total { background-color: #dcfce7; color: #166534; font-weight: bold; }
                            .grand-total { background-color: #fef9c3; color: #854d0e; font-size: 1.1em; font-weight: bold; }
                            @media print {
                                .no-print { display: none; }
                            }
                            .print-btn { padding: 10px 20px; font-size: 16px; cursor: pointer; margin-bottom: 20px; border-radius: 5px; border: none; background-color: #007bff; color: white; }
                        </style>
                    </head>
                    <body>
                        <div class="report-header">
                            <img src="${logoBase64}" alt="Logo">
                            <h1>${title}</h1>
                        </div>
                        <button class="no-print print-btn" onclick="window.print()">Imprimir ou Salvar como PDF</button>
                        <table>
                            ${tableContent}
                        </table>
                    </body>
                    </html>
                `;
            };

            const exportEntregasReport = () => {
                if (entries.length === 0) {
                    alert('Não há dados para exportar.');
                    return;
                }

                const sortedEntries = [...entries].sort((a, b) => new Date(a.date) - new Date(b.date));
                const groupedByMonth = sortedEntries.reduce((acc, entry) => {
                    const monthYear = new Date(entry.date + 'T00:00:00').toLocaleString('pt-BR', { month: 'long', year: 'numeric' });
                    if (!acc[monthYear]) {
                        acc[monthYear] = [];
                    }
                    acc[monthYear].push(entry);
                    return acc;
                }, {});

                let tableContent = '<thead><tr><th>Data</th><th>Plataforma</th><th>Valor da Entrega</th></tr></thead><tbody>';
                let grandTotal = 0;

                for (const month in groupedByMonth) {
                    tableContent += `<tr><td colspan="3" class="month-header">${month.charAt(0).toUpperCase() + month.slice(1)}</td></tr>`;
                    
                    let monthTotal = 0;
                    groupedByMonth[month].forEach(entry => {
                        const valor = entry.valorEntrega;
                        monthTotal += valor;
                        const formattedDate = new Date(entry.date + 'T00:00:00').toLocaleDateString('pt-BR');
                        tableContent += `<tr><td>${formattedDate}</td><td>${entry.platform}</td><td>${formatCurrency(valor)}</td></tr>`;
                    });

                    grandTotal += monthTotal;
                    tableContent += `<tr class="month-total"><td colspan="2">Total do Mês</td><td>${formatCurrency(monthTotal)}</td></tr>`;
                }
                
                tableContent += `<tr class="grand-total"><td colspan="2">Total Geral</td><td>${formatCurrency(grandTotal)}</td></tr>`;
                tableContent += '</tbody>';

                const reportHTML = getReportHTML('Relatório Mensal de Entregas', tableContent);
                const reportWindow = window.open('', '_blank');
                reportWindow.document.write(reportHTML);
                reportWindow.document.close();
            };

            const exportCustosReport = () => {
                if (entries.length === 0) {
                    alert('Não há dados para exportar.');
                    return;
                }

                const sortedEntries = [...entries].sort((a, b) => new Date(a.date) - new Date(b.date));
                const groupedByMonth = sortedEntries.reduce((acc, entry) => {
                    const monthYear = new Date(entry.date + 'T00:00:00').toLocaleString('pt-BR', { month: 'long', year: 'numeric' });
                    if (!acc[monthYear]) {
                        acc[monthYear] = [];
                    }
                    acc[monthYear].push(entry);
                    return acc;
                }, {});

                let tableContent = '<thead><tr><th>Categoria de Custo</th><th>Valor Total</th></tr></thead><tbody>';
                let grandTotal = 0;

                for (const month in groupedByMonth) {
                    tableContent += `<tr><td colspan="2" class="month-header">${month.charAt(0).toUpperCase() + month.slice(1)}</td></tr>`;
                    
                    const monthEntries = groupedByMonth[month];
                    const totalTaxa = monthEntries.reduce((sum, e) => sum + ((e.valorEntrega * e.taxaPlataforma) / 100), 0);
                    const totalCombustivel = monthEntries.reduce((sum, e) => sum + e.combustivel, 0);
                    const totalPedagio = monthEntries.reduce((sum, e) => sum + e.pedagio, 0);
                    const totalOutros = monthEntries.reduce((sum, e) => sum + e.outrosCustos, 0);
                    const monthTotal = totalTaxa + totalCombustivel + totalPedagio + totalOutros;
                    grandTotal += monthTotal;

                    if (totalTaxa > 0) tableContent += `<tr><td>Taxa da Plataforma</td><td>${formatCurrency(totalTaxa)}</td></tr>`;
                    if (totalCombustivel > 0) tableContent += `<tr><td>Combustível</td><td>${formatCurrency(totalCombustivel)}</td></tr>`;
                    if (totalPedagio > 0) tableContent += `<tr><td>Pedágio</td><td>${formatCurrency(totalPedagio)}</td></tr>`;
                    if (totalOutros > 0) tableContent += `<tr><td>Outros Custos</td><td>${formatCurrency(totalOutros)}</td></tr>`;
                    
                    tableContent += `<tr class="month-total"><td>Total do Mês</td><td>${formatCurrency(monthTotal)}</td></tr>`;
                }
                
                tableContent += `<tr class="grand-total"><td>Total Geral</td><td>${formatCurrency(grandTotal)}</td></tr>`;
                tableContent += '</tbody>';

                const reportHTML = getReportHTML('Relatório Mensal de Custos', tableContent);
                const reportWindow = window.open('', '_blank');
                reportWindow.document.write(reportHTML);
                reportWindow.document.close();
            };


            const renderAll = () => {
                renderTable();
                updateKPIs();
                updateCharts();
            };

            // Event Listeners
            addEntryBtn.addEventListener('click', () => {
                modalTitle.textContent = 'Adicionar Nova Corrida';
                document.getElementById('date').valueAsDate = new Date();
                showModal();
            });

            cancelBtn.addEventListener('click', hideModal);
            entryForm.addEventListener('submit', addOrUpdateEntry);
            exportEntregasBtn.addEventListener('click', exportEntregasReport);
            exportCustosBtn.addEventListener('click', exportCustosReport);
            
            entriesTableBody.addEventListener('click', (e) => {
                if (e.target.classList.contains('edit-btn')) {
                    const id = Number(e.target.dataset.id);
                    handleEdit(id);
                }
                if (e.target.classList.contains('delete-btn')) {
                    const id = Number(e.target.dataset.id);
                    deleteEntry(id);
                }
            });

            // Inicialização
            loadEntries();
            renderAll();
        });
    </script>
</body>
</html>
