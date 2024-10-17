<script>
    import { onMount } from "svelte";

    let charWidths = {};
    let spaceWidth = 0;
    let alphabet = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    // let sampleText = 'Lorem ipsum dolor sit amet consectetur adipisicing elit. One cat ran gown Roger turtle fish dog from amazon loose earn jam frog horse pidgeon wild old democratic kernal splice genetics koala';
    let lines = {
        line1: [],
        line2: []
    }

    let words = [];

    let char = '';

    function measureCharWidths(fontSize, fontFamily, letterSpacing = 0) {
        const span = document.createElement('span');
        span.style.fontSize = `${fontSize}px`;
        span.style.fontFamily = fontFamily;
        span.style.letterSpacing = `${letterSpacing}px`;
        span.style.position = 'absolute';
        span.style.visibility = 'hidden';
        span.style.whiteSpace = 'nowrap';
        document.body.appendChild(span);

        for (let char of alphabet) {
            span.textContent = char;
            charWidths[char] = span.offsetWidth;
        }

        document.body.removeChild(span);
    }

    function splitTextIntoLines(text, containerWidth) {
        const words = text.split(' ');
        let currentLine = '';
        let currentLineWidth = 0;
        lines = [];

        for (let word of words) {
            const wordWidth = calculateTextWidth(word + ' '); // Include space
            if (currentLineWidth + wordWidth <= containerWidth) {
                currentLine += word + ' ';
                currentLineWidth += wordWidth;
            } else {
                lines.push(currentLine.trim());
                currentLine = word + ' ';
                currentLineWidth = wordWidth;
            }
        }

        if (currentLine) {
            lines.push(currentLine.trim());
        }
    }

    function calculateTextWidth(text) {
        return text.split('').reduce((total, char) => total + (charWidths[char] || 0), 0);
    }

    function getCharWidth(char, fontSize, fontFamily, letterSpacing = 0) {
        // Create a temporary span element to measure the character width
        const span = document.createElement('span');
        span.textContent = char;

        // Set the font styles on the span
        span.style.fontSize = `${fontSize}px`;
        span.style.fontFamily = fontFamily;
        span.style.letterSpacing = `${letterSpacing}px`;
        span.style.position = 'absolute'; // Remove it from normal flow
        span.style.visibility = 'hidden'; // Hide it so it doesn't show on screen
        span.style.whiteSpace = 'nowrap'; // Prevent the span from wrapping

        // Append the span to the body to render it and measure it
        document.body.appendChild(span);

        // Get the width of the character
        const charWidth = span.offsetWidth;

        // Remove the span from the DOM
        document.body.removeChild(span);

        return charWidth;
    }


    function getSpaceWidth(fontSize, fontFamily, letterSpacing = 0) {
        // Helper function to create a span and measure its width
        const measureTextWidth = (text) => {
            const span = document.createElement('span');
            span.textContent = text;
            span.style.fontSize = `${fontSize}px`;
            span.style.fontFamily = fontFamily;
            span.style.letterSpacing = `${letterSpacing}px`;
            span.style.position = 'absolute'; // Remove it from normal flow
            span.style.visibility = 'hidden'; // Hide it so it doesn't show on screen
            span.style.whiteSpace = 'nowrap'; // Prevent wrapping

            // Append the span to the body to render it and measure it
            document.body.appendChild(span);

            // Get the width of the text
            const textWidth = span.offsetWidth;

            // Remove the span from the DOM
            document.body.removeChild(span);

            return textWidth;
        };

        // Measure "a a" (with a space) and "aa" (without a space)
        const widthWithSpace = measureTextWidth('a a');
        const widthWithoutSpace = measureTextWidth('aa');

        // The space width is the difference between the two measurements
        const spaceWidth = widthWithSpace - widthWithoutSpace;

        return spaceWidth;
    }

    function getRandomWords(words, containerWidth) {
        let currentLine = '';
        let currentLineWidth = 0;
        let selectedWords = [];

        while (words.length > 0) {
            const randomIndex = Math.floor(Math.random() * words.length);
            const word = words[randomIndex];
            const wordWidth = calculateTextWidth(word);

            if (currentLineWidth + wordWidth + (selectedWords.length > 0 ? spaceWidth : 0) <= containerWidth) {
                currentLine += word + ' ';
                currentLineWidth += wordWidth + (selectedWords.length > 0 ? spaceWidth : 0);
                selectedWords.push(word);
                words.splice(randomIndex, 1); // Remove the selected word from the array
            } else {
                break;
            }
        }

        return selectedWords;
    }


    onMount(async () => {
        try {
            const response = await fetch('/words.json');
            if (!response.ok) {
                throw new Error('Failed to fetch words.json');
            }
            const data = await response.json();

            console.log('Fetched data:', data);

            if (!Array.isArray(data.words)) {
                throw new Error('Fetched data does not contain an array of words');
            }

            words = data.words;

            const line = document.querySelector('.word-line');
            const computedStyle = window.getComputedStyle(line);
            const fontSize = parseFloat(computedStyle.fontSize);
            const fontFamily = computedStyle.fontFamily;
            const letterSpacingStr = computedStyle.letterSpacing;
            const letterSpacing = letterSpacingStr === 'normal' ? 0 : parseFloat(letterSpacingStr);

            measureCharWidths(fontSize, fontFamily, letterSpacing);
            spaceWidth = getSpaceWidth(fontSize, fontFamily, letterSpacing);

            const containerWidth = line.clientWidth;
            // Fill line1
            lines.line1 = getRandomWords([...words], containerWidth);
            // Fill line2
            lines.line2 = getRandomWords([...words], containerWidth);

            console.log('Lines:', lines.line1, lines.line2);
            console.log(charWidths);
            console.log(words);
        } catch (error) {
            console.error('Error loading words:', error);
        }
    });

</script>

<div id="test-ui">
    <div id="words">
        <div id="words-line-one" class="word-line">{lines.line1.join(' ')}</div>
        <div id="words-line-two" class="word-line">{lines.line2.join(' ')}</div>
    </div>
    <input type="text" id="text-input"> 
</div>


<style>
    #test-ui {
        font-size: 2rem;
        width: fit-content;
        padding: 20px;
        border: 1px solid black;
    }

    #words{
        font-family:'Times New Roman', Times, serif;
        width: 100%;
        margin: 0px 0px 20px 0px;
        border: 1px solid black;
    }

    .word-line {
        height: 50px;
        display: flex;
        flex-direction: row;
        align-items: center;

    }

    #text-input {
        width: 900px;
        font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        font-size: 2rem;
    }
</style>