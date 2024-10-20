<script>
    import { onMount } from "svelte";


    let lineRef;
    let computedStyle;
    let fontSize;
    let fontFamily;
    let letterSpacingStr;
    let letterSpacing;
    let paddingLeft;
    let paddingRight;
    let containerWidth;

    let charWidths = {};
    let spaceWidth = 0;
    let alphabet = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ-';
    // let sampleText = 'Lorem ipsum dolor sit amet consectetur adipisicing elit. One cat ran gown Roger turtle fish dog from amazon loose earn jam frog horse pidgeon wild old democratic kernal splice genetics koala';
    let lines = {
        line1: [],
        line2: []
    }

    let inputRef;

    let timer = 60;
    let interval;
    let WPM = 0;


    let words = [];

    let typedWordsLine = [];
    let typedWordsRecord = [];

    let char = '';
    let hasStartedTyping = false;

    let currentWord = ''
    let wordIndex = 0
    let lastKeyPressed = ''
    let currentWordInput = ''
    let isMatchingWord = false;

    let correctWordCounter = 0;

    function handleInput(event) {
        // Set the word input
        console.log('wordIndex:', wordIndex);
        currentWord = lines.line1[wordIndex];

        if (!hasStartedTyping) {
            hasStartedTyping = true;
            console.log('User has started typing');
            // Start timer
            startTimer();
        }

        if(timer > 0) {
            console.log('LastKeyPressed: ',lastKeyPressed);
            currentWordInput = inputRef.value;

            console.log('Current Word: ',currentWord);
            console.log('Current Input: ',currentWordInput);

            // Check if key pressed was space
            if(lastKeyPressed === ' ') {
                console.log('Space bar was pressed');

                // Check if user spelt word correctly
                isMatchingWord = currentWord === currentWordInput.trim();
                if(isMatchingWord) {
                    console.log('Word Matches: ', isMatchingWord);
                    console.log('User spelt word correctly');
                    correctWordCounter++;
                } else {
                    console.log('User spelt word incorrectly');
                }

                // Store results of typed words for current line
                typedWordsLine.push({
                    word: currentWord,
                    userSpelling: currentWordInput.trim(),
                    isCorrect: isMatchingWord
                });

                // store record of typed word
                typedWordsRecord.push({
                    word: currentWord,
                    userSpelling: currentWordInput.trim(),
                    isCorrect: isMatchingWord
                });

                calculateWPM();
                inputRef.value = ''; // Reset input field
                wordIndex++; // Increment word index

                // Check if index is out of bounds (end of line)
                if(wordIndex >= lines.line1.length) {
                    wordIndex = 0; // Reset index

                    typedWordsLine = []; // Clear array for new line

                    // Update to next line
                    updateLines();
                }

                console.log('Correct Word Count: ', correctWordCounter);
                console.log('Typed Words (Current Line): ', typedWordsLine);
                console.log('Typed Words (Total): ', typedWordsRecord);

                console.log('Lines: ', lines);
                

                updateSpanStyles();
            }
        } else {
            inputRef.value = '';
        }
    }

    function startTimer() {
        interval = setInterval(() => {
            timer--;
            if (timer <= 0) {
                clearInterval(interval);
                console.log('Time is up!');
                // Handle end of typing test
            }
        }, 1000);
    }

    function calculateWPM() {
        let timeInSeconds = 60 - timer
        let timeInMinutes = timeInSeconds / 60;
        WPM = correctWordCounter / timeInMinutes;
    }

    function updateLines() {
        lines = {
            line1: lines.line2,
            line2: getRandomWords([...words], containerWidth, paddingLeft, paddingRight)
        }
    }

    function updateSpanStyles() {
        console.log('Updating Spans on Line 1')
        const spans = document.querySelectorAll('#words-line-one span');
        spans.forEach((span, index) => {
            if (index === wordIndex) {
                span.style.backgroundColor = 'lightgrey';
            } else {
                span.style.backgroundColor = '';
            }
        });
    }

    function setInitalSpanStyles(line, index) {
    return line === 1 && index === wordIndex 
        ? `background-color: lightgrey; margin-right: ${spaceWidth / 2}px; margin-left: ${spaceWidth / 2}px;`
        : `background-color: white; margin-right: ${spaceWidth / 2}px; margin-left: ${spaceWidth / 2}px;`;
    }   

    function calculateTextWidth(text) {
        return text.split('').reduce((total, char) => total + (charWidths[char] || 0), 0);
    }

    function setSpanProperties(span, fontSize, fontFamily, letterSpacing) {
        span.style.fontSize = `${fontSize}px`;
        span.style.fontFamily = fontFamily;
        span.style.letterSpacing = `${letterSpacing}px`;
        span.style.position = 'absolute';
        span.style.visibility = 'hidden';
        span.style.whiteSpace = 'nowrap';
    }

    function measureCharWidths(fontSize, fontFamily, letterSpacing = 0) {
        const span = document.createElement('span');
        setSpanProperties(span, fontSize, fontFamily, letterSpacing);
        document.body.appendChild(span);

        for (let char of alphabet) {
            span.textContent = char;
            charWidths[char] = span.offsetWidth;
        }

        document.body.removeChild(span);
    }

    function getSpaceWidth(fontSize, fontFamily, letterSpacing = 0) {
        // Helper function to create a span and measure its width
        function measureTextWidth(text, fontSize, fontFamily, letterSpacing = 0) {
            const span = document.createElement('span');
            span.textContent = text;
            setSpanProperties(span, fontSize, fontFamily, letterSpacing);
            document.body.appendChild(span);
            const textWidth = span.offsetWidth;
            document.body.removeChild(span);
            return textWidth;
        }
        // Measure "a a" (with a space) and "aa" (without a space)
        const widthWithSpace = measureTextWidth('a a');
        const widthWithoutSpace = measureTextWidth('aa');
        // The space width is the difference between the two measurements
        const spaceWidth = widthWithSpace - widthWithoutSpace;
        return spaceWidth;
    }

    function getRandomWords(words, containerWidth, paddingLeft, paddingRight) {
        let currentLine = '';
        let currentLineWidth = 0;
        let selectedWords = [];
        const availableWidth = containerWidth - paddingLeft - paddingRight - 10;

        while (words.length > 0) {
            const randomIndex = Math.floor(Math.random() * words.length);
            const word = words[randomIndex];
            const wordWidth = calculateTextWidth(word);

            if (currentLineWidth + wordWidth + (selectedWords.length > 0 ? spaceWidth : 0) <= availableWidth) {
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

    async function fetchWords() {
        const response = await fetch('/words.json');
        if (!response.ok) {
            throw new Error('Failed to fetch words.json');
        }
        const data = await response.json();
        if (!Array.isArray(data.words)) {
            throw new Error('Fetched data does not contain an array of words');
        }
        return data.words;
    }



    onMount(async () => {
        try {
            words = await fetchWords();

            lineRef = document.querySelector('.word-line');
            computedStyle = window.getComputedStyle(lineRef);
            fontSize = parseFloat(computedStyle.fontSize);
            fontFamily = computedStyle.fontFamily;
            letterSpacingStr = computedStyle.letterSpacing;
            letterSpacing = letterSpacingStr === 'normal' ? 0 : parseFloat(letterSpacingStr);
            paddingLeft = parseFloat(computedStyle.paddingLeft);
            paddingRight = parseFloat(computedStyle.paddingRight);

            measureCharWidths(fontSize, fontFamily, letterSpacing);
            spaceWidth = getSpaceWidth(fontSize, fontFamily, letterSpacing);

            containerWidth = lineRef.clientWidth;
            // Fill line1
            lines.line1 = getRandomWords([...words], containerWidth, paddingLeft, paddingRight);
            // Fill line2
            lines.line2 = getRandomWords([...words], containerWidth, paddingLeft, paddingRight);

            const inputField = document.getElementById('text-input');
            inputField.addEventListener('keydown', (event) => {
                lastKeyPressed = event.key;
            });

            updateSpanStyles();

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
        <div id="words-line-one" class="word-line">
            {#each lines.line1 as word, index}
                <span style={setInitalSpanStyles(1, index)}>{word}</span>
            {/each}
        </div>
        <div id="words-line-two" class="word-line">
            {#each lines.line2 as word, index}
                <span style={setInitalSpanStyles(2, index)}>{word}</span>
            {/each}
        </div>
    </div>
    <input type="text" id="text-input"  bind:this={inputRef} on:input={handleInput}> 
    <div id="timer-display">Time left: {timer} seconds</div>
    <div>WPM: {WPM}</div>
</div>


<style>
    #test-ui {
        font-size: 2rem;
        width: fit-content;
        padding: 30px;
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
        padding: 0px 10px 0px 10px;
    }

    .word-line span {
        margin-right: 8px;
    }

    #text-input {
        width: 900px;
        font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        font-size: 2rem;
    }
</style>