// out: ..
<template lang="pug">
div(
  v-bind:style="computedStyle"
  style="position:absolute;display:block;box-sizing:border-box"
  v-if="opened"
  v-el:tt
  v-bind:class="computedClass"
  v-bind:id="id"
  )
  slot No content
</template>

<script lang="coffee">
module.exports =

  mixins:[
    require("vue-mixins/getViewportSize")
    require("vue-mixins/isOpened")
    require("vue-mixins/style")
    require("vue-mixins/class")
  ]

  props:
    "id":
      type: String
    "style":
      default: -> []
    "class":
      default: -> ["tooltip"]
    "anchor":
      type: String
      default: "snwe"
    "offset":
      type: Number
      default: 0
      coerce: Number
    "transitionIn":
      type: Function
      default: ({el,pos,style,cb}) ->
        for key,val of style
          @mergeStyle[key] = val
        cb()
    "transitionOut":
      type: Function
      default: ({el,style,cb}) ->
        for key,val of style
          @mergeStyle[key] = val
        cb()
    "ignoreParent":
      type: Boolean
      default: false
    "parent":
      type: Object
    "zIndex":
      type: Number
      default: 100
      coerce: Number
    "position":
      type: String
      default: "parent"

  data: ->
    mergeStyle:
      opacity: 0
      left: undefined
      top: undefined
      width: undefined
      height: undefined
      zIndex: @zIndex

  watch:
    "parent": "setupParent"
    "zIndex": (val) -> @mergeStyle.zIndex = val

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
        document.body.appendChild @$els.tt if @position == "body"
        parentPos = @parent.getBoundingClientRect()
        # get real pos depending on position in viewport
        windowSize = @getViewportSize()
        ttPos = @$els.tt.getBoundingClientRect()
        ttHeight = ttPos.height
        ttWidth = ttPos.width
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
        # get top/left relative to parent nw corner
        parentHeight = parentPos.height
        parentWidth = parentPos.width
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
        # positioning in document
        unless @position == "body"
          parentStyle = getComputedStyle(@parent)
          # correct border
          if parentStyle.getPropertyValue("box-sizing") == "border-box"
            top -= parseInt(parentStyle.getPropertyValue("border-top-width").replace("px",""))
            left -= parseInt(parentStyle.getPropertyValue("border-left-width").replace("px",""))
          isPositioned = /relative|absolute|fixed/.test(parentStyle.getPropertyValue("position"))
          unless isPositioned
            top += @parent.offsetTop
            left += @parent.offsetLeft
        else
          body = document.body
          docEl = document.documentElement
          scrollTop = window.pageYOffset || docEl.scrollTop || body.scrollTop
          scrollLeft = window.pageXOffset || docEl.scrollLeft || body.scrollLeft
          top += scrollTop + parentPos.top
          left += scrollLeft + parentPos.left
          isPositioned = false
        # set initial style before open
        if pos == "w" or pos == "e"
          @mergeStyle.top = top + 'px'
          @mergeStyle.left = if isPositioned then 0 else @parent.offsetLeft + 'px'
        else
          @mergeStyle.top = if isPositioned then 0 else @parent.offsetTop + 'px'
          @mergeStyle.left = left + 'px'
        @mergeStyle.height = ttHeight + 'px'
        @mergeStyle.width = ttWidth + 'px'
        # begin opening, wait for initial positioning for transition
        @$emit "beforeOpen"
        @$nextTick =>
          @transitionIn el:@$els.tt,pos:pos,style:{opacity:1,top:top+'px',left:left+'px'},cb: =>
            @$emit "opened"

    hide: ->
      return unless @opened
      @$emit "beforeClose"
      unless @position == "body"
        top = 0
        left = 0
        parentStyle = getComputedStyle(@parent)
        unless /relative|absolute|fixed/.test(parentStyle.getPropertyValue("position"))
          top += @parent.offsetTop
          left += @parent.offsetLeft
      else
        body = document.body
        docEl = document.documentElement
        scrollTop = window.pageYOffset || docEl.scrollTop || body.scrollTop
        scrollLeft = window.pageXOffset || docEl.scrollLeft || body.scrollLeft
        parentPos = @parent.getBoundingClientRect()
        top = scrollTop + parentPos.top
        left = scrollLeft + parentPos.left
      @transitionOut el:@$els.tt,style:{opacity:0,top:top+'px',left:left+'px'}, cb: =>
        @mergeStyle.width = undefined
        @mergeStyle.height = undefined
        @setClosed()
        @$emit "closed"
    toggle: ->
      if @opened
        @hide()
      else
        @show()
  ready: ->
    unless @parent?
      @parent = @$el.parentElement
    else
      @setupParent()

  beforeDestroy: ->
    @removeParentListener?()

</script>
