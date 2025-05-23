<script lang="jsx">
import { Transition } from "vue";
import { UNCHECKED, INDETERMINATE, CHECKED } from "../constants.js";
import { onLeftClick } from "../utils";
import Tip from "./Tip.vue";
import ArrowIcon from "./icons/Arrow.vue";

let arrowPlaceholder, checkMark, minusMark;

const Option = {
  name: "vue3-treeselect--option",
  inject: ["instance"],

  props: {
    node: {
      type: Object,
      required: true
    }
  },

  computed: {
    shouldExpand() {
      const { instance, node } = this;

      return node.isBranch && instance.shouldExpand(node);
    },

    shouldShow() {
      const { instance, node } = this;
      return instance.shouldShowOptionInMenu(node);
    }
  },

  methods: {
    renderOption() {
      const { instance, node } = this;
      const optionClass = {
        "vue3-treeselect__option": true,
        "vue3-treeselect__option--disabled": node.isDisabled,
        "vue3-treeselect__option--selected": instance.isSelected(node),
        "vue3-treeselect__option--highlight": node.isHighlighted,
        "vue3-treeselect__option--matched":
          instance.localSearch.active && node.isMatched,
        "vue3-treeselect__option--hide": !this.shouldShow
      };

      return (
        <div
          class={optionClass}
          onMouseenter={this.handleMouseEnterOption}
          data-id={node.id}>
          {this.renderArrow()}
          {this.renderLabelContainer([
            this.renderCheckboxContainer([this.renderCheckbox()]),
            this.renderLabel()
          ])}
        </div>
      );
    },

    renderSubOptionsList() {
      if (!this.shouldExpand) {
        return null;
      }

      return (
        <div class="vue3-treeselect__list">
          {this.renderSubOptions()}
          {this.renderNoChildrenTip()}
          {this.renderLoadingChildrenTip()}
          {this.renderLoadingChildrenErrorTip()}
        </div>
      );
    },

    renderArrow() {
      const { instance, node } = this;

      if (instance.shouldFlattenOptions && this.shouldShow) {
        return null;
      }
      if (node.isBranch) {
        const arrowClass = {
          "vue3-treeselect__option-arrow": true,
          "vue3-treeselect__option-arrow--rotated": this.shouldExpand
        };

        return (
          <div
            class="vue3-treeselect__option-arrow-container"
            onMousedown={this.handleMouseDownOnArrow}>
            <Transition
              name="vue3-treeselect__option-arrow--prepare"
              appear={true}>
              <ArrowIcon class={arrowClass} />
            </Transition>
          </div>
        );
      }

      // For leaf nodes, we render a placeholder to keep its label aligned to
      // branch nodes. Unless there is no branch nodes at all (a normal
      // non-tree select).
      if (/*node.isLeaf && */ instance.hasBranchNodes) {
        if (!arrowPlaceholder) {
          arrowPlaceholder = (
            <div class="vue3-treeselect__option-arrow-placeholder">&nbsp;</div>
          );
        }

        return arrowPlaceholder;
      }

      return null;
    },

    renderLabelContainer(children) {
      return (
        <div
          class="vue3-treeselect__label-container"
          onMousedown={this.handleMouseDownOnLabelContainer}>
          {children}
        </div>
      );
    },

    renderCheckboxContainer(children) {
      const { instance, node } = this;

      if (instance.single) {
        return null;
      }
      if (instance.disableBranchNodes && node.isBranch) {
        return null;
      }

      return <div class="vue3-treeselect__checkbox-container">{children}</div>;
    },

    renderCheckbox() {
      const { instance, node } = this;
      const checkedState = instance.forest.checkedStateMap[node.id];
      const checkboxClass = {
        "vue3-treeselect__checkbox": true,
        "vue3-treeselect__checkbox--checked": checkedState === CHECKED,
        "vue3-treeselect__checkbox--indeterminate":
          checkedState === INDETERMINATE,
        "vue3-treeselect__checkbox--unchecked": checkedState === UNCHECKED,
        "vue3-treeselect__checkbox--disabled": node.isDisabled
      };

      const customCheckboxRenderer = instance.$slots["option-checkbox"];

      if (customCheckboxRenderer) {
        return customCheckboxRenderer({
          node,
          checkboxClass,
          checkedState,
        });
      }

      if (!checkMark) {
        checkMark = <span class="vue3-treeselect__check-mark" />;
      }
      if (!minusMark) {
        minusMark = <span class="vue3-treeselect__minus-mark" />;
      }



      return (
        <span class={checkboxClass}>
          {checkMark}
          {minusMark}
        </span>
      );
    },

    renderLabel() {
      const { instance, node } = this;
      const shouldShowCount =
        node.isBranch &&
        (instance.localSearch.active
          ? instance.showCountOnSearchComputed
          : instance.showCount);
      const count = shouldShowCount
        ? instance.localSearch.active
          ? instance.localSearch.countMap[node.id][instance.showCountOf]
          : node.count[instance.showCountOf]
        : Number.NaN;
      const labelClassName = "vue3-treeselect__label";
      const countClassName = "vue3-treeselect__count";
      const customLabelRenderer = instance.$slots["option-label"];

      if (customLabelRenderer) {
        return customLabelRenderer({
          node,
          shouldShowCount,
          count,
          labelClassName,
          countClassName
        });
      }
      return (
        <label class={labelClassName}>
          {node.label}
          {shouldShowCount && <span class={countClassName}>({count})</span>}
        </label>
      );
    },

    renderSubOptions() {
      const { node } = this;

      if (!node.childrenStates.isLoaded) {
        return null;
      }

      return node.children.map((childNode) => (
        <Option node={childNode} key={childNode.id} />
      ));
    },

    renderNoChildrenTip() {
      const { instance, node } = this;

      if (!node.childrenStates.isLoaded || node.children.length) {
        return null;
      }

      return (
        <Tip type="no-children" icon="warning">
          {instance.noChildrenText}
        </Tip>
      );
    },

    renderLoadingChildrenTip() {
      const { instance, node } = this;

      if (!node.childrenStates.isLoading) {
        return null;
      }

      return (
        <Tip type="loading" icon="loader">
          {instance.loadingText}
        </Tip>
      );
    },

    renderLoadingChildrenErrorTip() {
      const { instance, node } = this;

      if (!node.childrenStates.loadingError) {
        return null;
      }

      return (
        <Tip type="error" icon="error">
          {node.childrenStates.loadingError}
          <a
            class="vue3-treeselect__retry"
            title={instance.retryTitle}
            onMousedown={this.handleMouseDownOnRetry}>
            {instance.retryText}
          </a>
        </Tip>
      );
    },

    handleMouseEnterOption(evt) {
      const { instance, node } = this;

      // Equivalent to `self` modifier.
      // istanbul ignore next
      if (evt.target !== evt.currentTarget) {
        return;
      }

      instance.setCurrentHighlightedOption(node, false);
    },

    handleMouseDownOnArrow: onLeftClick(
      function handleMouseDownOnOptionArrow() {
        const { instance, node } = this;
        instance.toggleExpanded(node);
      }
    ),

    handleMouseDownOnLabelContainer: onLeftClick(
      function handleMouseDownOnLabelContainer() {
        const { instance, node } = this;

        if (node.isBranch && instance.disableBranchNodes) {
          instance.toggleExpanded(node);
        } else {
          instance.select(node);
        }
      }
    ),

    handleMouseDownOnRetry: onLeftClick(function handleMouseDownOnRetry() {
      const { instance, node } = this;
      instance.loadChildrenOptions(node);
    })
  },

  render() {
    const { node } = this;
    const indentLevel = this.instance.shouldFlattenOptions ? 0 : node.level;
    const listItemClass = {
      "vue3-treeselect__list-item": true,
      [`vue3-treeselect__indent-level-${indentLevel}`]: true
    };

    return (
      <div class={listItemClass}>
        {this.renderOption()}
        {node.isBranch ? (
          <Transition name="vue3-treeselect__list--transition">
            {this.renderSubOptionsList()}
          </Transition>
        ) : (
          ""
        )}
      </div>
    );
  }
};

// eslint-disable-next-line vue/require-direct-export
export default Option;
</script>
