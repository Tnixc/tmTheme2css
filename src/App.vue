<template>
    <div class="container">
        <h1>.tmTheme to syntect css converter</h1>
        <div class="input-section">
            <textarea v-model="themeContent"
                placeholder="Paste your .tmTheme content here, or click the button below to upload"></textarea>
            <input type="file" @change="handleFileUpload" accept=".tmTheme">
        </div>
        <button @click="convertTheme" :disabled="!themeContent">Convert Theme</button>
        <div v-if="cssOutput" class="output-section">
            <h2>Generated CSS:</h2>
            <pre>{{ cssOutput }}</pre>
            <button @click="downloadCSS">Download CSS</button>
            <button @click="copyToClipboard">{{ copyButtonContent }}</button>
        </div>
        <div>
            <p>
                made by <a href="https://tnixc.space">tnixc</a> | source on <a
                    href="https://github.com/tnixc/tmTheme2css">github</a> |
                conversion handled by the <a href="https://github.com/Menci/syntect-js">syntect-js</a> library.
            </p>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import syntect from 'syntect';
import copy from 'clipboard-copy';

export default defineComponent({
    name: 'App',
    setup() {
        const themeContent = ref('');
        const cssOutput = ref('');
        const copyButtonContent = ref('Copy to Clipboard');

        const handleFileUpload = (event: Event) => {
            const file = (event.target as HTMLInputElement).files?.[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    themeContent.value = e.target?.result as string;
                };
                reader.readAsText(file);
            }
        };

        const convertTheme = async () => {
            const result = syntect.getCSS(themeContent.value, "");
            cssOutput.value = result.css; // Access the 'css' property of the result
            if (result.error) {
                alert(result.error);
            }
        };

        const copyToClipboard = () => {
            copy(cssOutput.value);
            copyButtonContent.value = "Copied!";
            setTimeout(() => {
                copyButtonContent.value = "Copy to Clipboard";
            }, 2000);
        };

        const downloadCSS = () => {
            const blob = new Blob([cssOutput.value], { type: 'text/css' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'theme.css';
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        };

        return {
            themeContent,
            cssOutput,
            handleFileUpload,
            convertTheme,
            copyToClipboard,
            copyButtonContent,
            downloadCSS,
        };
    },
});
</script>