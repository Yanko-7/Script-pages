<script setup>
import { ref } from 'vue'

const code_content = ref('')
const output_content = ref('')
function removeSubstrings(inputString) {
    // Remove occurrences of 'gme' substrings and replace certain keywords
    let res = inputString.replace(/"gme",/g, "");
    res = res.replace(/AcisOptions/g, "GmeOptions");
    res = res.replace(/ACIS_DELETE /g, "GME_DELETE ");
    res = res.replace(/ACIS_NEW /g, "GME_NEW ");
    res = res.replace(/ACIS_DELETE/g, "GME_DELETE");
    res = res.replace(/API_BEGIN/g, "__GME_API_BEGIN");
    res = res.replace(/API_END/g, "__GME_API_END");
    return res;
}

function processLine(content) {
    /**
     * Process a line of content.
     * - Removes 'gme_' prefixes unless followed by 'facet'.
     * - Removes 'gme' substrings.
     */
    if (content.includes('#include')) {
        return [content, false];
    }

    let flag = false;
    let processedContent = '';
    const contentLength = content.length;
    let index = 0;

    while (index < contentLength) {
        if (index + 4 < contentLength && content.substring(index, index + 4) === 'gme_') {
            if (index + 9 <= contentLength && content.substring(index + 4, index + 9) !== 'facet') {
                flag = true;
                index += 4;
                continue;
            }
        }
        processedContent += content[index];
        index++;
    }

    return [removeSubstrings(processedContent), flag];
}

function processFileContent(fileContent) {
    /**
     * Process a multi-line string content:
     * - Calls processLine function to process each line.
     */
    console.log(fileContent.value);
    const processedLines = [];
    const lines = fileContent.value.split(/\r?\n/);

    lines.forEach((line) => {
        const [newLine, _] = processLine(line);
        processedLines.push(newLine);
    });

    return processedLines;
}

// Example: Function to handle content from a textarea in the browser
function processTextareaContent() {
    //const textarea = document.getElementById('inputTextarea');
    //const fileContent = textarea.value;

    const processedLines = processFileContent(code_content);
    //const outputTextarea = document.getElementById('outputTextarea');
    output_content.value = processedLines.join('\n');
}



</script>

<template>
  <header>
    <!-- <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div> -->
  </header>

  <body class="flex items-center justify-center min-h-screen bg-gray-100">

    <div class="w-full h-[90vh] flex flex-col p-6 bg-white shadow rounded-lg">

      <div class="flex-grow flex">

        <textarea v-model="code_content" class="shadow textarea textarea-bordered w-1/2 h-full resize-none"
          placeholder="Enter text here..."></textarea>

        <div class="divider divider-vertical mx-4"></div>

        <textarea v-model="output_content" class="shadow textarea textarea-bordered w-1/2 h-full resize-none"
          placeholder="output text here..."></textarea>

      </div>

      <div class="mt-6 text-center">
        <button @click="processTextareaContent" class="btn  w-40">Submit</button>
      </div>

    </div>

  </body>
</template>

<style scoped></style>
