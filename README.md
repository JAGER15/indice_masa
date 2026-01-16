# indice_masa
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Sistema IMC - Versión Estática</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        .form {
            margin-bottom: 30px;
        }
        .form_label {
            display: block;
            margin-top: 10px;
            font-weight: bold;
        }
        .form_input {
            width: 100%;
            padding: 8px;
            margin-top: 5px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
        }
        button {
            margin-top: 15px;
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            margin-right: 10px;
        }
        button:hover {
            background-color: #45a049;
        }
        h1 {
            color: #333;
            border-bottom: 2px solid #4CAF50;
            padding-bottom: 10px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: left;
        }
        th {
            background-color: #4CAF50;
            color: white;
        }
        tr:nth-child(even) {
            background-color: #f2f2f2;
        }
        .imc-low { background-color: #ffeb3b; }
        .imc-normal { background-color: #8bc34a; }
        .imc-overweight { background-color: #ff9800; }
        .imc-obesity { background-color: #f44336; color: white; }
        .btn-modify { background-color: #2196F3; }
        .btn-delete { background-color: #f44336; }
        .btn-back { background-color: #607D8B; }
        .actions a, .actions button {
            padding: 5px 10px;
            color: white;
            text-decoration: none;
            border: none;
            border-radius: 4px;
            margin-right: 5px;
            cursor: pointer;
            font-size: 14px;
        }
        .actions button {
            background: none;
        }
        .footer {
            margin-top: 20px;
            text-align: center;
        }
        .alert {
            padding: 15px;
            margin-bottom: 20px;
            border: 1px solid transparent;
            border-radius: 4px;
        }
        .alert-success {
            color: #3c763d;
            background-color: #dff0d8;
            border-color: #d6e9c6;
        }
        .alert-error {
            color: #a94442;
            background-color: #f2dede;
            border-color: #ebccd1;
        }
        .hidden {
            display: none;
        }
        .action-buttons {
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Formulario principal (mostrado por defecto) -->
        <div id="formSection">
            <form id="imcForm" class="form">
                <h1>Datos de la persona</h1>
                <label class="form_label">ID</label>
                <input class="form_input" type="text" id="id" name="id" required>
                <label class="form_label">Nombre</label>
                <input class="form_input" type="text" id="nombre" name="nombre" required>
                <label class="form_label">Peso (kg)</label>
                <input class="form_input" type="number" step="0.01" id="peso" name="peso" required>
                <label class="form_label">Altura (m)</label>
                <input class="form_input" type="number" step="0.01" id="altura" name="altura" required>
                <button type="submit">GUARDAR</button>
                <button type="button" id="showDataBtn">MOSTRAR DATOS</button>
            </form>
            <div id="message" class="alert hidden"></div>
        </div>

        <!-- Tabla para mostrar datos -->
        <div id="tableSection" class="hidden">
            <h1>Lista de Personas</h1>
            <div class="action-buttons">
                <button onclick="showForm()">Agregar Nueva Persona</button>
                <button onclick="exportToJSON()">Exportar a JSON</button>
                <button onclick="importFromJSON()">Importar desde JSON</button>
            </div>
            
            <div class="table-container">
                <table id="dataTable">
                    <thead>
                        <tr>
                            <th>ID</th>
                            <th>Nombre</th>
                            <th>Peso (kg)</th>
                            <th>Altura (m)</th>
                            <th>IMC</th>
                            <th>Estado</th>
                            <th>Consejo</th>
                            <th>Acciones</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody">
                        <!-- Los datos se cargan aquí con JavaScript -->
                    </tbody>
                </table>
            </div>
            
            <div class="footer">
                <button class="btn-back" onclick="showForm()">Regresar al Formulario</button>
            </div>
        </div>

        <!-- Formulario de modificación (oculto inicialmente) -->
        <div id="modifySection" class="hidden">
            <h1>Modificar datos</h1>
            <form id="modifyForm">
                <input type="hidden" id="modifyId">
                <label class="form_label">Nombre:</label>
                <input class="form_input" type="text" id="modifyNombre" required><br><br>
                <label class="form_label">Peso (kg):</label>
                <input class="form_input" type="number" step="0.01" id="modifyPeso" required><br><br>
                <label class="form_label">Altura (m):</label>
                <input class="form_input" type="number" step="0.01" id="modifyAltura" required><br><br>

                <button type="submit">Guardar cambios</button>
                <button type="button" onclick="showTable()">Cancelar</button>
            </form>
        </div>
    </div>

    <!-- Input oculto para importar JSON -->
    <input type="file" id="jsonFileInput" accept=".json" style="display: none;">

    <script>
        // Almacenamiento en localStorage del navegador
        const STORAGE_KEY = 'imc_data';
        
        // Cargar datos al iniciar
        let personas = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
        
        // Mostrar formulario principal
        function showForm() {
            document.getElementById('formSection').classList.remove('hidden');
            document.getElementById('tableSection').classList.add('hidden');
            document.getElementById('modifySection').classList.add('hidden');
            document.getElementById('imcForm').reset();
        }
        
        // Mostrar tabla de datos
        function showTable() {
            document.getElementById('formSection').classList.add('hidden');
            document.getElementById('tableSection').classList.remove('hidden');
            document.getElementById('modifySection').classList.add('hidden');
            loadTableData();
        }
        
        // Calcular IMC y determinar estado
        function calcularIMC(peso, altura) {
            return peso / (altura * altura);
        }
        
        function determinarEstado(imc) {
            if (imc < 18.5) {
                return {
                    estado: "Bajo peso",
                    consejo: "Debes aumentar de peso. Consulta a un nutricionista.",
                    clase: "imc-low"
                };
            } else if (imc >= 18.5 && imc < 25) {
                return {
                    estado: "Peso normal",
                    consejo: "¡Excelente! Mantén tus hábitos saludables.",
                    clase: "imc-normal"
                };
            } else if (imc >= 25 && imc < 30) {
                return {
                    estado: "Sobrepeso",
                    consejo: "Necesitas hacer ejercicio y mejorar tu alimentación.",
                    clase: "imc-overweight"
                };
            } else {
                return {
                    estado: "Obesidad",
                    consejo: "Consulta a un médico y nutricionista urgentemente.",
                    clase: "imc-obesity"
                };
            }
        }
        
        // Guardar nueva persona
        document.getElementById('imcForm').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const id = document.getElementById('id').value;
            const nombre = document.getElementById('nombre').value;
            const peso = parseFloat(document.getElementById('peso').value);
            const altura = parseFloat(document.getElementById('altura').value);
            
            // Verificar si el ID ya existe
            if (personas.some(p => p.id === id)) {
                showMessage('Error: El ID ya existe. Use otro ID.', 'error');
                return;
            }
            
            const nuevaPersona = { id, nombre, peso, altura };
            personas.push(nuevaPersona);
            localStorage.setItem(STORAGE_KEY, JSON.stringify(personas));
            
            showMessage('Datos guardados correctamente!', 'success');
            document.getElementById('imcForm').reset();
        });
        
        // Cargar datos en la tabla
        function loadTableData() {
            const tableBody = document.getElementById('tableBody');
            tableBody.innerHTML = '';
            
            if (personas.length === 0) {
                tableBody.innerHTML = `
                    <tr>
                        <td colspan="8" style="text-align:center;">
                            No hay datos registrados. Agrega una persona primero.
                        </td>
                    </tr>
                `;
                return;
            }
            
            personas.forEach(persona => {
                const imc = calcularIMC(persona.peso, persona.altura);
                const estadoInfo = determinarEstado(imc);
                
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${persona.id}</td>
                    <td>${persona.nombre}</td>
                    <td>${persona.peso.toFixed(2)} kg</td>
                    <td>${persona.altura.toFixed(2)} m</td>
                    <td class="${estadoInfo.clase}">${imc.toFixed(2)}</td>
                    <td class="${estadoInfo.clase}">${estadoInfo.estado}</td>
                    <td>${estadoInfo.consejo}</td>
                    <td>
                        <div class="actions">
                            <button class="btn-modify" onclick="modifyPerson('${persona.id}')">Modificar</button>
                            <button class="btn-delete" onclick="deletePerson('${persona.id}')">Eliminar</button>
                        </div>
                    </td>
                `;
                tableBody.appendChild(row);
            });
        }
        
        // Modificar persona
        function modifyPerson(id) {
            const persona = personas.find(p => p.id === id);
            if (!persona) return;
            
            document.getElementById('modifyId').value = persona.id;
            document.getElementById('modifyNombre').value = persona.nombre;
            document.getElementById('modifyPeso').value = persona.peso;
            document.getElementById('modifyAltura').value = persona.altura;
            
            document.getElementById('formSection').classList.add('hidden');
            document.getElementById('tableSection').classList.add('hidden');
            document.getElementById('modifySection').classList.remove('hidden');
        }
        
        // Guardar modificación
        document.getElementById('modifyForm').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const id = document.getElementById('modifyId').value;
            const nombre = document.getElementById('modifyNombre').value;
            const peso = parseFloat(document.getElementById('modifyPeso').value);
            const altura = parseFloat(document.getElementById('modifyAltura').value);
            
            const index = personas.findIndex(p => p.id === id);
            if (index !== -1) {
                personas[index] = { id, nombre, peso, altura };
                localStorage.setItem(STORAGE_KEY, JSON.stringify(personas));
                showTable();
                showMessage('Datos modificados correctamente!', 'success');
            }
        });
        
        // Eliminar persona
        function deletePerson(id) {
            if (confirm('¿Seguro que deseas eliminar a esta persona?')) {
                personas = personas.filter(p => p.id !== id);
                localStorage.setItem(STORAGE_KEY, JSON.stringify(personas));
                loadTableData();
                showMessage('Persona eliminada correctamente!', 'success');
            }
        }
        
        // Exportar a JSON
        function exportToJSON() {
            const dataStr = JSON.stringify(personas, null, 2);
            const dataUri = 'data:application/json;charset=utf-8,'+ encodeURIComponent(dataStr);
            
            const exportFileDefaultName = 'imc_data.json';
            
            const linkElement = document.createElement('a');
            linkElement.setAttribute('href', dataUri);
            linkElement.setAttribute('download', exportFileDefaultName);
            linkElement.click();
            
            showMessage('Datos exportados a JSON correctamente!', 'success');
        }
        
        // Importar desde JSON
        function importFromJSON() {
            document.getElementById('jsonFileInput').click();
        }
        
        document.getElementById('jsonFileInput').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = function(event) {
                try {
                    const importedData = JSON.parse(event.target.result);
                    
                    // Validar estructura
                    if (!Array.isArray(importedData)) {
                        throw new Error('El archivo no contiene un array válido');
                    }
                    
                    // Combinar datos (evitar duplicados por ID)
                    importedData.forEach(item => {
                        if (!personas.some(p => p.id === item.id)) {
                            personas.push(item);
                        }
                    });
                    
                    localStorage.setItem(STORAGE_KEY, JSON.stringify(personas));
                    loadTableData();
                    showMessage('Datos importados correctamente!', 'success');
                    
                    // Resetear input
                    document.getElementById('jsonFileInput').value = '';
                } catch (error) {
                    showMessage('Error al importar: ' + error.message, 'error');
                }
            };
            reader.readAsText(file);
        });
        
        // Mostrar mensajes
        function showMessage(text, type) {
            const messageDiv = document.getElementById('message');
            messageDiv.textContent = text;
            messageDiv.className = `alert alert-${type}`;
            messageDiv.classList.remove('hidden');
            
            setTimeout(() => {
                messageDiv.classList.add('hidden');
            }, 3000);
        }
        
        // Event Listeners
        document.getElementById('showDataBtn').addEventListener('click', showTable);
        
        // Inicializar
        showForm();
    </script>
</body>
</html>
