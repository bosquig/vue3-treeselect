<script>
  import { h } from 'vue'
  import Input from './Input'
  import Placeholder from './Placeholder'

  export default {
    name: 'vue-treeselect--single-value',
    inject: [ 'instance' ],
    methods: {
      renderSingleValueLabel() {
        const { instance } = this
        const node = instance.selectedNodes[0]

        const customValueLabelRenderer = instance.$slots['value-label']
        return customValueLabelRenderer
          ? customValueLabelRenderer({ node })
          : node.label
      },
    },
    render() {
      const { instance, $parent: { renderValueContainer } } = this
      const shouldShowValue = instance.hasValue && !instance.trigger.searchQuery
      const children = [
          h(
              Placeholder
          ),
          h(
              Input,
              {
                ref: 'input',
              }
          )
      ]

      if (shouldShowValue) {
        children.unshift(h(
            'div',
            {
              class: 'vue-treeselect__single-value',
            },
            [
                this.renderSingleValueLabel(),
            ]
        ))
      }

      return renderValueContainer(children)
    },
  }
</script>
