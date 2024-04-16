<script>
  import { h } from 'vue'
  import { onLeftClick } from '../utils'
  import DeleteIcon from './icons/Delete'

  export default {
    name: 'vue-treeselect--multi-value-item',
    inject: [ 'instance' ],

    props: {
      node: {
        type: Object,
        required: true,
      },
    },

    methods: {
      handleMouseDown: onLeftClick(function handleMouseDown() {
        const { instance, node } = this

        // Deselect this node.
        instance.select(node)
      }),
    },

    render() {
      const { instance, node } = this
      const itemClass = {
        'vue-treeselect__multi-value-item': true,
        'vue-treeselect__multi-value-item-disabled': node.isDisabled,
        'vue-treeselect__multi-value-item-new': node.isNew,
      }
      const customValueLabelRenderer = instance.$slots['value-label']
      const labelRenderer = customValueLabelRenderer ? customValueLabelRenderer({ node }) : node.label

      return h(
          'div',
          {
            class: 'vue-treeselect__multi-value-item-container',
          },
          [
              h(
                  'div',
                  {
                    class: itemClass,
                    onMousedown: this.handleMouseDown,
                  },
                  [
                      h(
                          'span',
                          {
                            class: 'vue-treeselect__multi-value-label',
                          },
                          labelRenderer
                      ),
                      h(
                          'span',
                          {
                            class: 'vue-treeselect__icon vue-treeselect__value-remove',
                          },
                          [
                              h(DeleteIcon)
                          ]
                      )
                  ]
              )
          ]
      )
    },
  }
</script>
