const allFoodWords = [
    'PIZZA', 'PAELLA', 'SUSHI', 'TACOS', 'ENSALADA', 'SOPA', 'ARROZ', 'PASTA', 'POLLO', 'CARNE',
    'PESCADO', 'QUESO', 'LECHE', 'JUGO', 'VINO', 'CEBOLLA', 'AJO', 'VAINILLA', 'AZUCAR', 'HARINA',
    'HUEVOS', 'MANZANA', 'NARANJA', 'FRESAS', 'CHOCOLATE', 'GALLETAS', 'PANQUEQUE', 'TARTA', 'MANTECA', 'SALSA'
];

const GRID_SIZE = 16; // Cuadrícula de 16x16
const NUM_WORDS = 15; // 15 palabras por partida
const wordSearchGrid = document.getElementById('wordSearchGrid');
const wordListElement = document.getElementById('wordList');
const newGameButton = document.getElementById('newGameButton');

let selectedWords = [];
let grid = [];
let isMouseDown = false;
let startCell = null;
let currentSelection = []; // Celdas seleccionadas temporalmente
let foundWords = new Set(); // Para llevar un registro de las palabras encontradas

function initializeGame() {
    selectedWords = getRandomWords(allFoodWords, NUM_WORDS);
    grid = createEmptyGrid(GRID_SIZE);
    placeWordsInGrid(selectedWords, grid);
    fillEmptyCells(grid);
    renderGrid(grid);
    renderWordList(selectedWords);

    // Reiniciar estados
    isMouseDown = false;
    startCell = null;
    currentSelection = [];
    foundWords = new Set();
}

function getRandomWords(wordPool, count) {
    const shuffled = [...wordPool].sort(() => 0.5 - Math.random());
    return shuffled.slice(0, count).sort(); // Ordenar alfabéticamente para la lista
}

function createEmptyGrid(size) {
    return Array(size).fill(0).map(() => Array(size).fill(''));
}

function placeWordsInGrid(words, grid) {
    words.forEach(word => {
        let placed = false;
        let attempts = 0;
        const maxAttempts = 100; // Evitar bucles infinitos

        while (!placed && attempts < maxAttempts) {
            const row = Math.floor(Math.random() * GRID_SIZE);
            const col = Math.floor(Math.random() * (GRID_SIZE - word.length + 1)); // Solo horizontal

            // Verificar si la palabra cabe y no choca con letras existentes no vacías
            let canPlace = true;
            for (let i = 0; i < word.length; i++) {
                if (grid[row][col + i] !== '' && grid[row][col + i] !== word[i]) {
                    canPlace = false;
                    break;
                }
            }

            if (canPlace) {
                for (let i = 0; i < word.length; i++) {
                    grid[row][col + i] = word[i];
                }
                placed = true;
            }
            attempts++;
        }
        // Si no se pudo colocar, es un problema de diseño o palabras muy largas para la cuadrícula
        if (!placed) {
            console.warn(`No se pudo colocar la palabra: ${word}`);
        }
    });
}

function fillEmptyCells(grid) {
    const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    for (let r = 0; r < GRID_SIZE; r++) {
        for (let c = 0; c < GRID_SIZE; c++) {
            if (grid[r][c] === '') {
                grid[r][c] = alphabet[Math.floor(Math.random() * alphabet.length)];
            }
        }
    }
}

function renderGrid(grid) {
    wordSearchGrid.innerHTML = '';
    wordSearchGrid.style.setProperty('--grid-cols', GRID_SIZE); // Establecer variable CSS
    grid.forEach((row, rowIndex) => {
        row.forEach((cell, colIndex) => {
            const div = document.createElement('div');
            div.classList.add('grid-cell');
            div.dataset.row = rowIndex;
            div.dataset.col = colIndex;
            div.textContent = cell;
            wordSearchGrid.appendChild(div);

            div.addEventListener('mousedown', handleMouseDown);
            div.addEventListener('mouseup', handleMouseUp);
            div.addEventListener('mouseenter', handleMouseEnter);
        });
    });
}

function renderWordList(words) {
    wordListElement.innerHTML = '';
    words.forEach(word => {
        const li = document.createElement('li');
        const checkbox = document.createElement('div');
        checkbox.classList.add('checkbox');
        checkbox.innerHTML = '&nbsp;'; // Espacio para que no esté vacío

        const wordText = document.createElement('span');
        wordText.textContent = word;

        li.appendChild(checkbox);
        li.appendChild(wordText);
        li.dataset.word = word; // Guardar la palabra en el dataset
        wordListElement.appendChild(li);
    });
}

function handleMouseDown(e) {
    isMouseDown = true;
    startCell = e.target;
    clearSelection();
    e.target.classList.add('selected');
    currentSelection.push(e.target);
}

function handleMouseUp() {
    isMouseDown = false;
    if (currentSelection.length > 0) {
        const selectedWordText = currentSelection.map(cell => cell.textContent).join('');
        checkWord(selectedWordText);
    }
    clearSelection();
}

function handleMouseEnter(e) {
    if (!isMouseDown || !startCell) return;

    const targetCell = e.target;
    const startRow = parseInt(startCell.dataset.row);
    const startCol = parseInt(startCell.dataset.col);
    const targetRow = parseInt(targetCell.dataset.row);
    const targetCol = parseInt(targetCell.dataset.col);

    // Solo permitir selección horizontal de izquierda a derecha
    if (startRow === targetRow && targetCol >= startCol) {
        clearSelection(); // Limpiar antes de calcular la nueva selección
        for (let i = startCol; i <= targetCol; i++) {
            const cell = wordSearchGrid.querySelector(`[data-row="${startRow}"][data-col="${i}"]`);
            if (cell) {
                cell.classList.add('selected');
                currentSelection.push(cell);
            }
        }
    } else {
        // Si no es una selección válida, simplemente marcamos la celda de inicio
        clearSelection();
        startCell.classList.add('selected');
        currentSelection = [startCell];
    }
}


function clearSelection() {
    currentSelection.forEach(cell => cell.classList.remove('selected'));
    currentSelection = [];
    startCell = null;
}

function checkWord(word) {
    if (selectedWords.includes(word) && !foundWords.has(word)) {
        // Marcar las celdas de la cuadrícula como encontradas (permanente)
        currentSelection.forEach(cell => {
            cell.classList.remove('selected'); // Eliminar selección temporal
            cell.classList.add('found'); // Añadir clase de encontrado
        });

        // Marcar la palabra en la lista inferior (automático con checkbox)
        const listItem = wordListElement.querySelector(`li[data-word="${word}"]`);
        if (listItem) {
            listItem.classList.add('found-word');
            const checkbox = listItem.querySelector('.checkbox');
            checkbox.classList.add('checked');
            checkbox.innerHTML = '&#10003;'; // Símbolo de verificación
        }
        foundWords.add(word); // Añadir a la lista de palabras encontradas
    }
}

newGameButton.addEventListener('click', initializeGame);

// Iniciar el juego la primera vez
initializeGame();