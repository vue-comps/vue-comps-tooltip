// out: ..
<template lang="jade">
div(:style="style" v-if="opened" v-el:tt v-bind:class="[class]")
  slot No content
</template>

<script lang="coffee">
module.exports =

  mixins:[
    require("vue-mixins/getViewportSize")
    require("vue-mixins/isOpened")
  ]

  props:
    "class":
      type: String
      default: "tooltip"
    "anchor":
      type: String
      default: "snwe"
    "offset":
      type: Number
      default: 0
    "transitionIn":
      type: Function
      default: ({el,pos,style,cb}) ->
        for key,val of style
          @style[key] = val
        cb()
    "transitionOut":
      type: Function
      default: ({el,style,cb}) ->
        for key,val of style
          @style[key] = val
        cb()
    "ignoreParent":
      type: Boolean
      default: false
    "parent":
      type: Object
    "zIndex":
      type: Number
      default: 100

  data: ->
    removeParentListener: null
    removeTimeout: null
    style:
      position: "absolute"
      opacity: 0
      left: undefined
      top: undefined
      width: undefined
      height: undefined
      display: "block"
      zIndex: @zIndex
      boxSizing: "border-box"

  watch:
    "parent": "setupParent"
    "zIndex": (val) -> @style.zIndex = val

  methods:
    setupParent: (parent = @parent) ->
      unless @ignoreParent
        @removeParentListener?()
        parent.addEventListener "mouseenter", @show
        parent.addEventListener "mouseleave", @hide
        @removeParentListener = =>
          parent.removeEventListener "mouseenter", @show
          parent.removeEventListener "mouseleave", @hide
    show: ->
      return if @opened
      @setOpened()
      @$nextTick =>
        parentPos = @parent.getBoundingClientRect()
        windowSize = @getViewportSize()
        ttHeight = @$els.tt.offsetHeight
        ttWidth = @$els.tt.offsetWidth
        parentHeight = @parent.offsetHeight
        parentWidth = @parent.offsetWidth
        @style.height = ttHeight + 'px'
        @style.width = ttWidth + 'px'
        height = ttHeight + @offset
        width = ttWidth + @offset
        pos = null
        for a in @anchor
          pos = a
          if a == "s" and parentPos.bottom + height <= windowSize.height
            break
          else if a == "n" and parentPos.top - height >= 0
            break
          else if a == "w" and parentPos.left - width >= 0
            break
          else if a == "e" and parentPos.right + width <= windowSize.width
            break

        if pos == "s"
          top = parentHeight + @offset
          left = parentWidth/2 - ttWidth/2
        else if pos == "n"
          top = - height
          left = parentWidth/2 - ttWidth/2
        else if pos == "w"
          top = parentHeight/2 - ttHeight/2
          left = - width
        else if pos == "e"
          top = parentHeight/2 - ttHeight/2
          left = parentWidth + @offset

        parentStyle = getComputedStyle(@parent)
        if parentStyle.getPropertyValue("box-sizing") == "border-box"
          top -= parseInt(parentStyle.getPropertyValue("border-top-width").replace("px",""))
          left -= parseInt(parentStyle.getPropertyValue("border-left-width").replace("px",""))
        isPositioned = /relative|absolute|fixed/.test(parentStyle.getPropertyValue("position"))
        unless isPositioned
          top += @parent.offsetTop
          left += @parent.offsetLeft
        if pos == "w" or pos == "e"
          @style.top = top + 'px'
          @style.left = if isPositioned then 0 else @parent.offsetLeft + 'px'
        else
          @style.top = if isPositioned then 0 else @parent.offsetTop + 'px'
          @style.left = left + 'px'



        @$emit "beforeOpen"
        @$nextTick =>
          @transitionIn el:@$els.tt,pos:pos,style:{opacity:1,top:top+'px',left:left+'px'},cb: =>
            @$emit "opened"

    hide: ->
      return unless @opened
      @$emit "beforeClose"
      top = 0
      left = 0
      parentStyle = getComputedStyle(@parent)
      unless /relative|absolute|fixed/.test(parentStyle.getPropertyValue("position"))
        top += @parent.offsetTop
        left += @parent.offsetLeft
      @transitionOut el:@$els.tt,style:{opacity:0,top:top+'px',left:left+'px'}, cb: =>
        @style.width = undefined
        @style.height = undefined
        @setClosed()
        @$emit "closed"
    toggle: ->
      if @opened
        @hide()
      else
        @show()
  attached: ->
    unless @parent?
      @parent = @$el.parentElement
    else
      @setupParent()

  dettached: ->
    @removeParentListener?()

</script>
