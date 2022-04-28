<script>
  import { h } from 'vue'
  import { UNCHECKED, INDETERMINATE, CHECKED } from '../constants'
  import { onLeftClick } from '../utils'
  import Tip from './Tip'
  import ArrowIcon from './icons/Arrow'

  let arrowPlaceholder, checkMark, minusMark

  const Option = {
    name: 'vue-treeselect--option',
    inject: [ 'instance' ],

    props: {
      node: {
        type: Object,
        required: true,
      },
    },

    computed: {
      shouldExpand() {
        const { instance, node } = this

        return node.isBranch && instance.shouldExpand(node)
      },

      shouldShow() {
        const { instance, node } = this
        return instance.shouldShowOptionInMenu(node)
      },
    },

    methods: {
      renderOption() {
        const { instance, node } = this
        const optionClass = {
          'vue-treeselect__option': true,
          'vue-treeselect__option--disabled': node.isDisabled,
          'vue-treeselect__option--selected': instance.isSelected(node),
          'vue-treeselect__option--highlight': node.isHighlighted,
          'vue-treeselect__option--matched': instance.localSearch.active && node.isMatched,
          'vue-treeselect__option--hide': !this.shouldShow,
        }

        return h(
            'div',
            {
              class: optionClass,
              onMouseenter: this.handleMouseEnterOption,
              'data-id': node.id,
            },
            [
                this.renderArrow(),
                this.renderLabelContainer([
                    this.renderCheckboxContainer([
                        this.renderCheckbox(),
                    ]),
                    this.renderLabel(),
                ])
            ]
        )
      },

      renderSubOptionsList() {
        if (!this.shouldExpand) return null

        return h(
            'div',
            {
              class: 'vue-treeselect__list',
            },
            [
              this.renderSubOptions(),
              this.renderNoChildrenTip(),
              this.renderLoadingChildrenTip(),
              this.renderLoadingChildrenErrorTip(),
            ]
        )
      },

      renderArrow() {
        const { instance, node } = this

        if (instance.shouldFlattenOptions && this.shouldShow) return null
        if (node.isBranch) {

          const arrowClass = {
            'vue-treeselect__option-arrow': true,
            'vue-treeselect__option-arrow--rotated': this.shouldExpand,
          }

          return h(
              'div',
              {
                class: 'vue-treeselect__option-arrow-container',
                onMousedown: this.handleMouseDownOnArrow,
              },
              [
                  h(
                      'transition',
                      {
                        name: 'vue-treeselect__option-arrow--prepare',
                        appear: true,
                      },
                      [
                          h(
                              ArrowIcon,
                              {
                                class: arrowClass,
                              }
                          )
                      ]
                  )
              ]
          )
        }

        // For leaf nodes, we render a placeholder to keep its label aligned to
        // branch nodes. Unless there is no branch nodes at all (a normal
        // non-tree select).
        if (/*node.isLeaf && */instance.hasBranchNodes) {
          if (!arrowPlaceholder) arrowPlaceholder = h(
              'div',
              {
                class: 'vue-treeselect__option-arrow-placeholder',
                innerHTML: '&nbsp;',
              }
          )

          return arrowPlaceholder
        }

        return null
      },

      renderLabelContainer(children) {
        return h(
            'div',
            {
              class: 'vue-treeselect__label-container',
              onMousedown: this.handleMouseDownOnLabelContainer,
            },
            children
        )
      },

      renderCheckboxContainer(children) {
        const { instance, node } = this

        if (instance.single) return null
        if (instance.disableBranchNodes && node.isBranch) return null

        return h(
            'div',
            {
              class: 'vue-treeselect__checkbox-container',
            },
            children
        )
      },

      renderCheckbox() {
        const { instance, node } = this
        const checkedState = instance.forest.checkedStateMap[node.id]
        const checkboxClass = {
          'vue-treeselect__checkbox': true,
          'vue-treeselect__checkbox--checked': checkedState === CHECKED,
          'vue-treeselect__checkbox--indeterminate': checkedState === INDETERMINATE,
          'vue-treeselect__checkbox--unchecked': checkedState === UNCHECKED,
          'vue-treeselect__checkbox--disabled': node.isDisabled,
        }

        if (!checkMark) checkMark = h(
            'span',
            {
              class: 'vue-treeselect__check-mark',
            }
        )
        if (!minusMark) minusMark = h(
            'span',
            {
              class: 'vue-treeselect__minus-mark',
            }
        )

        return h(
            'span',
            {
              class: checkboxClass,
            },
            [
                checkMark,
                minusMark,
            ]
        )
      },

      renderLabel() {
        const { instance, node } = this
        const shouldShowCount = (
          node.isBranch && (instance.localSearch.active
            ? instance.showCountOnSearchComputed
            : instance.showCount
          )
        )
        const count = shouldShowCount
          ? instance.localSearch.active
            ? instance.localSearch.countMap[node.id][instance.showCountOf]
            : node.count[instance.showCountOf]
          : NaN
        const labelClassName = 'vue-treeselect__label'
        const countClassName = 'vue-treeselect__count'
        const customLabelRenderer = instance.$slots['option-label']

        if (customLabelRenderer) return customLabelRenderer({
          node,
          shouldShowCount,
          count,
          labelClassName,
          countClassName,
        })

        const children = []

        if (shouldShowCount) {
          children.push((
              'span',
                  {
                    class: countClassName,
                    text: '(' + count + ')',
                  }
          ))
        }

        return h(
            'label',
            {
              class: labelClassName,
              text: node.label,
            },
            children
        )
      },

      renderSubOptions() {
        const { node } = this
        
        if (!node.childrenStates.isLoaded) return null

        return node.children.map(childNode => {
          return h(
              'vue-treeselect--option',
              {
                node: childNode,
                key: childNode.id,
              }
          )
        })
      },

      renderNoChildrenTip() {
        const { instance, node } = this

        if (!node.childrenStates.isLoaded || node.children.length) return null

        return h(
            Tip,
            {
              type: 'no-children',
              icon: 'warning',
              text: instance.noChildrenText,
            }
        )
      },

      renderLoadingChildrenTip() {
        const { instance, node } = this

        if (!node.childrenStates.isLoading) return null

        return h(
            Tip,
            {
              type: 'loading',
              icon: 'loader',
              text: instance.loadingText,
            }
        )
      },

      renderLoadingChildrenErrorTip() {
        const { instance, node } = this

        if (!node.childrenStates.loadingError) return null

        return h(
            Tip,
            {
              type: 'error',
              icon: 'error',
              text: node.childrenStates.loadingError,
            },
            [
                h(
                    'a',
                    {
                      class: 'vue-treeselect__retry',
                      title: instance.retryTitle,
                      text: instance.retryText,
                      onMousedown: this.handleMouseDownOnRetry,
                    }
                )
            ]
        )
      },

      handleMouseEnterOption(evt) {
        const { instance, node } = this

        // Equivalent to `self` modifier.
        // istanbul ignore next
        if (evt.target !== evt.currentTarget) return

        instance.setCurrentHighlightedOption(node, false)
      },

      handleMouseDownOnArrow: onLeftClick(function handleMouseDownOnOptionArrow() {
        const { instance, node } = this
        instance.toggleExpanded(node)
      }),

      handleMouseDownOnLabelContainer: onLeftClick(function handleMouseDownOnLabelContainer() {
        const { instance, node } = this

        if (node.isBranch && instance.disableBranchNodes) {
          instance.toggleExpanded(node)
        } else {
          instance.select(node)
        }
      }),

      handleMouseDownOnRetry: onLeftClick(function handleMouseDownOnRetry() {
        const { instance, node } = this
        instance.loadChildrenOptions(node)
      }),
    },

    render() {
      const { node } = this
      const indentLevel = this.instance.shouldFlattenOptions ? 0 : node.level
      const listItemClass = {
        'vue-treeselect__list-item': true,
        [`vue-treeselect__indent-level-${indentLevel}`]: true,
      }

      const children = [
        this.renderOption(),
      ]

      if (node.isBranch) {
        children.push(h(
            'transition',
            {
              name: 'vue-treeselect__list--transition',
            },
            [
                this.renderSubOptionsList(),
            ]
        ))
      }

      return h(
          'div',
          {
            class: listItemClass,
          },
          [
              this.renderOption(),
          ]
      )
    },
  }

  // eslint-disable-next-line vue/require-direct-export
  export default Option
</script>