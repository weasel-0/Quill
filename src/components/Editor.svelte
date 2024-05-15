<script>
    import { onMount, onDestroy } from 'svelte'
    import { Editor } from '@tiptap/core'
    import StarterKit from '@tiptap/starter-kit'
    import CodeBlockLowlight from '@tiptap/extension-code-block-lowlight'
    import Image from '@tiptap/extension-image'
    import Link from '@tiptap/extension-link'
    import Typography from '@tiptap/extension-typography'
    import Placeholder from '@tiptap/extension-placeholder'
    import Document from '@tiptap/extension-document'
    import html2pdf from 'html2pdf.js'
    import { theme } from '$root/stores/themeStorage.js'

    import { common, createLowlight } from 'lowlight'

    let element
    let editor
    const CustomDocument = Document.extend({
        content: 'heading block*',
    })

    onMount(() => {
        editor = new Editor({
            element: element,
            autofocus: true,
            extensions: [
                CustomDocument,
                StarterKit.configure({
                    codeBlock: false,
                    document: false,
                }),
                CodeBlockLowlight.configure({
                    lowlight: createLowlight(common),
                }),
                Image,
                Link.configure({
                    openOnClick: true,
                    autolink: true,
                }),
                Link.extend({
                    inclusive: false,
                }),
                Typography,
                Placeholder.configure({
                    placeholder: 'Write something …',
                }),
            ],
            // content: '<p>Hello World! 🌍️ </p>',
            onTransaction: () => {
                // force re-render so `editor.isActive` works as expected
                editor = editor
            },
        })
    })

    onDestroy(() => {
        if (editor) {
            editor.destroy()
        }
    })

    function exportMe() {
        const html = editor.getHTML()
        const elements = document.getElementsByClassName('tiptap')
        console.log(elements)
        const opt = {
            margin: [12, 12, 12, 12],
            filename: 'myfile.pdf',
            image: { type: 'jpeg', quality: 0.2 },
            html2canvas: { scale: 2, useCORS: true },
            jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' },
            pagebreak: { mode: 'avoid-all', before: '#page2el' },
        }

        let combinedHtml = ''
        for (let i = 0; i < elements.length; i++) {
            combinedHtml += elements[i].outerHTML
        }

        // html2pdf(combinedHtml)
        html2pdf().set(opt).from(combinedHtml).save()
    }

    /*for theme*/

    function toggleTheme() {
        if ($theme === 'light') {
            theme.set('dark')
            localStorage.setItem('theme', 'dark')
        } else {
            theme.set('light')
            localStorage.setItem('theme', 'light')
        }
    }

    $: document.body.className = $theme
</script>

{#if editor}
    <button on:click={toggleTheme}
        >{$theme.charAt(0).toUpperCase() + $theme.slice(1)}</button
    >
    <button on:click={exportMe} class="exportBtn" title="Export to pdf"
        ><svg
            xmlns="http://www.w3.org/2000/svg"
            width="32"
            height="32"
            fill="#000000"
            viewBox="0 0 256 256"
            ><path
                d="M200,164v8h12a12,12,0,0,1,0,24H200v12a12,12,0,0,1-24,0V152a12,12,0,0,1,12-12h32a12,12,0,0,1,0,24ZM92,172a32,32,0,0,1-32,32H56v4a12,12,0,0,1-24,0V152a12,12,0,0,1,12-12H60A32,32,0,0,1,92,172Zm-24,0a8,8,0,0,0-8-8H56v16h4A8,8,0,0,0,68,172Zm100,8a40,40,0,0,1-40,40H112a12,12,0,0,1-12-12V152a12,12,0,0,1,12-12h16A40,40,0,0,1,168,180Zm-24,0a16,16,0,0,0-16-16h-4v32h4A16,16,0,0,0,144,180ZM36,108V40A20,20,0,0,1,56,20h96a12,12,0,0,1,8.49,3.52l56,56A12,12,0,0,1,220,88v20a12,12,0,0,1-24,0v-4H148a12,12,0,0,1-12-12V44H60v64a12,12,0,0,1-24,0ZM160,57V80h23Z"
            ></path></svg
        ></button
    >
{/if}
<div bind:this={element} class="editor" />

<style>
    .editor {
        /* color: #444; */
        font-size: 18px;
        margin: 40px auto;
        /* max-width: 650px; */
        max-width: 980px;
    }

    .exportBtn {
        background-color: #fff;
        border: 0;
        border-radius: 5px;
        position: fixed;
        bottom: 20px;
        right: 20px;
        padding: 8px;
    }
    .exportBtn:hover {
        cursor: pointer;
        background-color: #eee;
        transition: 0.2s ease-in;
    }
</style>
