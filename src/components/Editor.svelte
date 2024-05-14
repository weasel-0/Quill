<script>
    import { onMount, onDestroy } from 'svelte'
    import { Editor } from '@tiptap/core'
    import StarterKit from '@tiptap/starter-kit'
    import CodeBlockLowlight from '@tiptap/extension-code-block-lowlight'
    import Image from '@tiptap/extension-image'
    import Link from '@tiptap/extension-link'

    import { common, createLowlight } from 'lowlight'

    let element
    let editor

    onMount(() => {
        editor = new Editor({
            element: element,
            autofocus: true,
            extensions: [
                StarterKit.configure({
                    codeBlock: false,
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
</script>

<div bind:this={element} class="editor" />

<style>
    .editor {
        color: #444;
        font-size: 18px;
        margin: 40px auto;
        /* max-width: 650px; */
        max-width: 980px;
    }
</style>
