<template>
  <div ref="page"></div>
</template>

<script>
import { TextLayerBuilder } from 'pdfjs-dist/web/pdf_viewer'
import 'pdfjs-dist/web/pdf_viewer.css'

export default {
  name: 'Page',
  props: {
    page: {
      type: Object,
      default: () => ({}),
      required: true
    },
    scale: {
      type: Number,
      default: 1
    },
    isRenderText: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      canvas: null,
      textLayerDiv: null,
      pdfText: null
    }
  },
  mounted () {
    this.renderPage()
  },
  methods: {
    renderPage () {
      const viewport = this.page.getViewport({ scale: this.scale })
      const pageContainer = document.createElement('div')
      const canvas = document.createElement('canvas')
      canvas.height = viewport.height
      canvas.width = viewport.width
      const canvasContext = canvas.getContext('2d')
      const renderContext = {
        canvasContext,
        viewport
      }
      this.canvas = canvas

      pageContainer.setAttribute('id', `page-${this.page.pageIndex + 1}`)
      pageContainer.classList.add('page-container')
      pageContainer.appendChild(canvas)

      this.page.render(renderContext)
      this.$refs.page.appendChild(pageContainer)

      this.isRenderText && this.renderText(this.page, viewport, pageContainer)
    },
    sizeChange () {
      // handle canvas change
      this.canvas.classList.add('temp-status')
      this.textLayerDiv.classList.add('temp-status')

      const viewport = this.page.getViewport({ scale: this.scale })
      this.canvas.height = viewport.height
      this.canvas.width = viewport.width
      const canvasContext = this.canvas.getContext('2d')
      const renderContext = {
        canvasContext,
        viewport
      }

      this.page.render(renderContext)
      this.canvas.style.visibility = 'visible'

      if (!this.isRenderText) return
      // handle text change
      this.textLayerDiv.innerHTML = ''
      this.textLayerDiv.setAttribute('style', `width: ${viewport.width}px; margin: 0 auto;`)
      const textLayer = new TextLayerBuilder({
        textLayerDiv: this.textLayerDiv,
        pageIndex: this.page.pageIndex,
        viewport
      })

      textLayer.setTextContent(this.pdfText)
      textLayer.render()

      this.canvas.classList.remove('temp-status')
      this.textLayerDiv.classList.remove('temp-status')
    },
    async renderText (page, viewport, container) {
      page.getTextContent()
        .then(content => {
          const textLayerDiv = document.createElement('div')
          textLayerDiv.setAttribute('class', 'textLayer')
          textLayerDiv.setAttribute('style', `width: ${viewport.width}px; margin: 0 auto;`)
          const textLayer = new TextLayerBuilder({
            textLayerDiv,
            pageIndex: page.pageIndex,
            viewport
          })

          this.textLayerDiv = textLayerDiv
          this.pdfText = content

          textLayer.setTextContent(this.pdfText)
          textLayer.render()
          container.appendChild(textLayerDiv)
        })
        .catch(err => {
          throw new Error('render pdf text error::', err)
        })
    }
  },
  watch: {
    scale (val) {
      this.sizeChange()
    }
  }
}
</script>

<style>
.page-container {
  position: relative;
  display: flex;
  justify-content: center;
}
.temp-status {
  position: absolute;
  display: none;
}
.finish-status {
  position: relative;
  display: block;
}
</style>
