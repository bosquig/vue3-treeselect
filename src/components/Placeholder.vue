<script>
  import { h } from 'vue'

  export default {
    name: 'vue-treeselect--placeholder',
    inject: [ 'instance' ],

    render() {
      const { instance } = this
      const placeholderClass = {
        'vue-treeselect__placeholder': true,
        'vue-treeselect-helper-zoom-effect-off': true,
        'vue-treeselect-helper-hide': instance.hasValue || instance.trigger.searchQuery,
      }

      return h('div',
          {
            class: placeholderClass,
          },
          instance.placeholder
      )
    },
  }
</script>
