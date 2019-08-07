<script lang="jsx">
  import Sortable from 'sortablejs'
  import render from './render'
  import { createNamespacedHelpers } from 'vuex'

  const { mapGetters, mapState, mapMutations, mapActions } = createNamespacedHelpers('project')
  //
  // const SuitsHandlerDirections = [
  //   '↖', '↑', '↗',
  //   '←', '→',
  //   '↙', '↓', '↘'
  // ]

  const SuitsHandlerDirections = [
      '↑',
    '←', '→',
      '↓'
  ]

  let dropId = '0'

  let previewDrag = false

  export default {
    name: 'comp-suit',
    props: {
      uid: {
        type: String
      },
      nodeName: {
        type: String
      },
      node: {
        type: Object
      }
    },
    data () {
      return {
        $sortable: null,
        isOver: false,
        isDragging: false
      }
    },
    computed: {
      ...mapState([
        'activeUid'
      ]),
      $dom () {
        let $el = this.$el
        // fixme: element-ui的$el做了代理不是根$el
        if (this.node.name === 'ElInput') {
          $el = this.$el.parentNode
        }
        return $el
      },
      classes () {
        return {
          'comp-suit': true,
          '--active': this.isActive,
          '--over': this.isOver,
          '--isHanding': this.isHanding,
          '--isContainer': this.isContainer,
          '--isLocked': this.isLocked
        }
      },
      nodeUid () {
        return this.node.uid
      },
      isRoot () {
        return this.nodeUid === 'root'
      },
      isContainer () {
        const compData = this.$store.getters['library/getComponent'](this.node.name)
        return compData.isContainer !== undefined ? compData.isContainer : false
      },
      isActive () {
        return this.activeUid === this.nodeUid
      },
      isLocked () {
        return false
        // return this.ActiveNode.uiConfig.locked
      }
    },
    methods: {
      ...mapActions([
        'sortNode',
        'insertNode'
      ]),
      ...mapMutations([
        'SET_ACTIVE_NODE'
      ]),
      initSuits () {
        // 组件边框
        SuitsHandlerDirections.forEach(direction => {
          const $dom = document.createElement('div')
          $dom.classList.add('comp-suit-handler')
          $dom.classList.add(`--${direction}`)
          this.$el.appendChild($dom)
        })
        // 组件遮罩
        const $dom = document.createElement('div')
        $dom.classList.add('comp-suit-mask')
        this.$el.appendChild($dom)
      },
      init () {
        this.initSuits()

        this.$sortable = new Sortable(this.$dom, {
          group: this.nodeUid,
          draggable: '.comp-suit',
          // Element is dropped into the list from another list
          // Changed sorting within list
          onUpdate: this.handleDragUpdate,
          onSort: this.handleDragSort
        })
      },
      handleDragSort (e) {
        console.log(this.node.pid)
        console.log(e.oldDraggableIndex)
        console.log(e.newDraggableIndex)
        // if (this.node.pid) {
        //   this.sortNode({
        //     id: this.node.pid,
        //     oldIndex: e.oldDraggableIndex,
        //     newIndex: e.newDraggableIndex
        //   })
        // }
      },
      handleDragUpdate (e) {
      },
      handleActive (e) {
        this.SET_ACTIVE_NODE(this.nodeUid)
        e.stopPropagation()
      },
      handleDragStart (e) {
        previewDrag = true
      },
      handleDragEnd (e) {
        previewDrag = false
      },
      handleDragOver (e) {
        if (!previewDrag && this.isContainer) {
          this.SET_ACTIVE_NODE(this.nodeUid)
          e.stopPropagation()
        }
      },
      handleInsertNode (val) {
        this.insertNode({
          name: val.name,
          pid: this.activeUid
        })
      },
      addSuitListeners () {
        // document.addEventListener('mousemove', this.handleMouseMove)
        // document.addEventListener('mouseup', this.handleMouseUp)
        this.$el.addEventListener('mousedown', this.handleActive)
        this.$el.addEventListener('dragover', this.handleDragOver)
        if (this.isRoot) {
          this.$el.addEventListener('dragstart', this.handleDragStart)
          this.$el.addEventListener('dragend', this.handleDragEnd)
          this.eventBus.$on('insertNode', this.handleInsertNode)
        }
        // this.$el.addEventListener('dragleave', this.handleDragLeave)
      },
      removeSuitListeners () {
        // document.removeEventListener('mousemove', this.handleMouseMove)
        // document.removeEventListener('mouseup', this.handleMouseUp)
        this.$el.removeEventListener('mousedown', this.handleMouseDown)
        this.$el.removeEventListener('dragover', this.handleDragOver)
        if (this.isRoot) {
          this.$el.removeEventListener('dragstart', this.handleDragStart)
          this.$el.removeEventListener('dragend', this.handleDragEnd)
          this.eventBus.$off('insertNode', this.handleInsertNode)
        }
        // this.$el.removeEventListener('dragleave', this.handleDragLeave)
      }
    },
    mounted () {
      this.init()
      this.addSuitListeners()
    },
    beforeDestroy () {
      this.removeSuitListeners()
    },
    render (h) {
      this.node.nodeData.class = this.node.nodeData.class ? {
        [`uid-${this.node.uid}`]: true,
        ...this.node.nodeData.class,
        ...this.classes
      } : {
        [`uid-${this.node.uid}`]: true,
        ...this.classes
      }
      // this.node.nodeData.directives = [
      //   {
      //     name: 'drag-ani'
      //   }
      // ]
      // this.node.nodeData.attrs = this.node.nodeData.attrs === undefined ? {
      //   draggable: true,
      // } : {
      //   draggable: true,
      //   ...this.node.nodeData.attrs
      // }
      return h(
        this.node.name,
        this.node.nodeData,
        this.node.children ? render(h, this.node.children) : []
      )
    }
  }
</script>
<style lang="scss">
  @import "~@mobov/scss-helper/import";
  // $--element-active-color: #ffd0a3;
  $--comp-suit-handler-color-normal: #03a9f4;
  $--comp-suit-handler-color-locked: #ff5252;
  $--comp-suit-handler-color-over: #ffa365;
  $--comp-suit-handler-pos-fix: 0;
  $--comp-suit-handler-size: 2px;
  $--comp-suit-handler-position: -1px;
  $--comp-suit-handler-corner-size: 10px;
  // $--comp-suit-select-border-size: 2px;

  .comp-suit {
    --comp-suit-handler-color: #{$--comp-suit-handler-color-normal};
    --comp-suit-handler-scale: .5;
    user-select: none;
    // pointer-events: none;
    box-sizing: border-box;
    position: relative;
    transform: translate3d(0, 0, 0);
    &.--active {
      z-index: 99;
      /*border-width: 0 !important;*/
      >.comp-suit-handler {
        background-color: var(--comp-suit-handler-color);
        /*transition: transform ease .3s;*/
        position: absolute;
      }
      >.--↑,
      >.--→,
      >.--↓,
      >.--← {
        z-index: 100;
      }
      >.--↑,
      >.--↓ {
        height: $--comp-suit-handler-size;
        width: calc(100% + #{$--comp-suit-handler-size }) ;
        left: $--comp-suit-handler-position;
      }

      >.--→,
      >.--← {
        width: $--comp-suit-handler-size;
        height: calc(100% + #{$--comp-suit-handler-size }) ;
        top: $--comp-suit-handler-position;
      }

      >.--↑ {
        top: $--comp-suit-handler-position;
      }

      >.--↓ {
        bottom: $--comp-suit-handler-position;
      }

      >.--← {
        left: $--comp-suit-handler-position;
      }

      >.--→ {
        right: $--comp-suit-handler-position;
      }
    }
    &.--isContainer {
      border: 1px dashed rgba(0, 0, 0, .5);
      >.comp-suit-mask {
        display: none;
      }
    }
    &.--isLocked {
      --comp-suit-handler-color: #{$--comp-suit-handler-color-locked};
    }
    &.--isHanding {
      --comp-suit-handler-scale: 1
    }
    .comp-suit-mask {
      /*visibility: hidden;*/
      background-color: transparent;
      cursor: grab;
      height: 100%;
      width: 100%;
      position: absolute;
      left: 0;
      top: 0;
      z-index: 1;
    }
  }

  .previewer {
    &:hover {
      .comp-suit {
        --comp-suit-handler-scale: 1
      }
    }
  }

  /*.comp-suit-axis {*/
    /*width: 1px;*/
    /*height: 100vh;*/
    /*position: absolute;*/
    /*background-color: var(--comp-suit-handler-color);*/
  /*}*/

</style>