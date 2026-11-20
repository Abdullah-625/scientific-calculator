
/* ================================
   1. VARIABLES
================================ */

const display = document.getElementById("display");
const previous = document.getElementById("previous");
const angleMode = document.getElementById("angle-mode");
const mode = document.getElementById("mode");
const historyPanel = document.getElementById("history-panel");
const historyList = document.getElementById("history-list");

let expression = "";
let answer = 0;
let isRadians = false;
let secondMode = false;
let justCalculated = false;


/* ================================
   2. DISPLAY
================================ */

function updateDisplay() {
    display.textContent = expression || "0";
    mode.textContent = isRadians ? "RAD" : "DEG";
    angleMode.textContent = isRadians ? "RAD" : "DEG";
}


/* ================================
   3. INPUT HANDLING
================================ */

function addValue(value) {

    if (justCalculated && /[0-9.(]/.test(value)) {
        expression = "";
    }

    justCalculated = false;

    if (value === "pi") {
        expression += "pi";
    } else if (value === "e") {
        expression += "e";
    } else if (value === "ans") {
        expression += "ans";
    } else {
        expression += value;
    }

    updateDisplay();
}


/* ================================
   4. SCIENTIFIC FUNCTIONS
================================ */

function addFunction(fn) {

    if (justCalculated) {
        justCalculated = false;
    }

    const functions = {
        sin: secondMode ? "asin(" : "sin(",
        cos: secondMode ? "acos(" : "cos(",
        tan: secondMode ? "atan(" : "tan(",
        log: "log(",
        ln: "ln(",
        sqrt: "sqrt(",
        square: "^2",
        factorial: "!",
    };

    expression += functions[fn] || fn;

    secondMode = false;
    updateDisplay();
}


/* ================================
   5. MATHEMATICAL ENGINE
================================ */

function factorial(n) {

    if (!Number.isInteger(n) || n < 0 || n > 170) {
        throw new Error("Invalid factorial");
    }

    let result = 1;

    for (let i = 2; i <= n; i++) {
        result *= i;
    }

    return result;
}


function calculateExpression(input) {

    let exp = input;

    // Replace constants
    exp = exp.replace(/\bpi\b/g, "Math.PI");
    exp = exp.replace(/\be\b/g, "Math.E");
    exp = exp.replace(/\bans\b/g, `(${answer})`);

    // Replace scientific functions
    exp = exp.replace(/\bsin\(/g, "Math.sin(");
    exp = exp.replace(/\bcos\(/g, "Math.cos(");
    exp = exp.replace(/\btan\(/g, "Math.tan(");

    exp = exp.replace(/\basin\(/g, "Math.asin(");
    exp = exp.replace(/\bacos\(/g, "Math.acos(");
    exp = exp.replace(/\batan\(/g, "Math.atan(");

    exp = exp.replace(/\bsqrt\(/g, "Math.sqrt(");
    exp = exp.replace(/\blog\(/g, "Math.log10(");
    exp = exp.replace(/\bln\(/g, "Math.log(");

    // Convert degrees to radians
    if (!isRadians) {

        exp = exp.replace(
            /Math\.(sin|cos|tan)\(/g,
            "Math.$1((Math.PI/180)*"
        );

        // Convert inverse trig results to degrees
        exp = exp.replace(
            /Math\.(asin|acos|atan)\(/g,
            "(180/Math.PI)*Math.$1("
        );
    }

    // Convert factorial
    exp = exp.replace(
        /(\d+(?:\.\d+)?|\([^()]*\))!/g,
        "factorial($1)"
    );

    // Convert percentages
    exp = exp.replace(/(\d+(?:\.\d+)?|\([^()]*\))%/g, "($1/100)");

    // Convert powers
    exp = exp.replace(/\^/g, "**");

    // Validate mathematical characters
    if (!/^[0-9+\-*/().,\sA-Za-z_*]+$/.test(exp)) {
        throw new Error("Invalid expression");
    }

    // Evaluate expression
    const result = Function(
        "factorial",
        `"use strict"; return (${exp});`
    )(factorial);

    if (!Number.isFinite(result)) {
        throw new Error("Math error");
    }

    return result;
}


/* ================================
   6. CALCULATE
================================ */

function calculate() {

    if (!expression) return;

    try {

        const originalExpression = expression;
        const result = calculateExpression(expression);

        answer = result;

        previous.textContent = originalExpression + " =";

        expression = Number(result.toPrecision(12)).toString();

        justCalculated = true;

        addHistory(originalExpression, expression);

        updateDisplay();

    } catch (error) {

        display.textContent = "Error";
        expression = "";
        justCalculated = false;

    }
}


/* ================================
   7. HISTORY
================================ */

function addHistory(exp, result) {

    const item = document.createElement("div");
    item.className = "history-item";

    const calculation = document.createElement("span");
    calculation.textContent = exp;

    const output = document.createElement("strong");
    output.textContent = "= " + result;

    item.append(calculation, output);

    historyList.prepend(item);
}


/* ================================
   8. BUTTON ACTIONS
================================ */

function clearCalculator() {

    expression = "";
    previous.textContent = "";
    justCalculated = false;

    updateDisplay();
}


function deleteLast() {

    if (justCalculated) {
        clearCalculator();
        return;
    }

    expression = expression.slice(0, -1);

    updateDisplay();
}


/* ================================
   9. BUTTON EVENTS
================================ */

document.querySelectorAll("[data-value]").forEach(button => {

    button.addEventListener("click", () => {
        addValue(button.dataset.value);
    });

});


document.querySelectorAll("[data-fn]").forEach(button => {

    button.addEventListener("click", () => {
        addFunction(button.dataset.fn);
    });

});


document.querySelectorAll("[data-action]").forEach(button => {

    button.addEventListener("click", () => {

        const action = button.dataset.action;

        if (action === "clear") clearCalculator();
        if (action === "delete") deleteLast();
        if (action === "calculate") calculate();

    });

});


/* ================================
   10. ANGLE MODE
================================ */

angleMode.addEventListener("click", () => {

    isRadians = !isRadians;

    updateDisplay();

});


/* ================================
   11. SECOND MODE
================================ */

document.getElementById("second").addEventListener("click", function () {

    secondMode = !secondMode;

    this.style.background = secondMode ? "#4a8cda" : "#30394b";

});


/* ================================
   12. HISTORY CONTROLS
================================ */

document.getElementById("history-toggle").addEventListener("click", () => {

    historyPanel.classList.toggle("hidden");

});


document.getElementById("clear-history").addEventListener("click", () => {

    historyList.innerHTML = '<p class="empty-history">No calculations yet</p>';

});


/* ================================
   13. KEYBOARD SUPPORT
================================ */

document.addEventListener("keydown", event => {

    const key = event.key;

    if (/^[0-9.+\-*/^()%!]$/.test(key)) {
        addValue(key);
    }

    else if (key === "Enter" || key === "=") {
        event.preventDefault();
        calculate();
    }

    else if (key === "Backspace") {
        deleteLast();
    }

    else if (key === "Escape") {
        clearCalculator();
    }

});


/* ================================
   14. INITIALIZE
================================ */

updateDisplay();
