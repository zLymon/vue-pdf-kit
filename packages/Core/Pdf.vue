<template>
  <div id="pdf-root"></div>
</template>

<script>
import PDFJS from 'pdfjs-dist'

export default {
  name: 'Pdf',
  props: {
    sourceData: {
      type: [String, Object],
      default: '',
      required: true
    },
    isLoaded: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      pageData: []
    }
  },
  mounted () {
    this.getDocument()
  },
  computed: {
    validate () {
      if (typeof this.sourceData === 'string') {
        return this.sourceData !== ''
      } else if (typeof this.sourceData === 'object') {
        return Object.keys(this.sourceData).length !== 0
      } else {
        return false
      }
    }
  },
  methods: {
    getDocument () {
      if (!this.validate) {
        throw new Error('Please set a valid url or data of pdf')
      }

      PDFJS.getDocument(this.sourceData).promise
        .then(async pdf => {
          const totalPage = pdf.numPages

          const getPageTask = []
          for (let i = 1; i <= totalPage; i++) {
            getPageTask.push(pdf.getPage(i))
          }

          Promise.all(getPageTask)
            .then(list => {
              this.pageData = list
              this.$emit('update:is-loaded', true)
              this.$emit('handlePageData', this.pageData)
            })
        })
        .catch(err => {
          throw new Error(err)
        })
    }
  }
}
</script>
