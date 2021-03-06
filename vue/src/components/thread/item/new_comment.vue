<style lang="scss">
</style>

<script lang="coffee">
import Session        from '@/shared/services/session'
import EventBus       from '@/shared/services/event_bus'
import AbilityService from '@/shared/services/ability_service'
import LmoUrlService  from '@/shared/services/lmo_url_service'
import FlashService   from '@/shared/services/flash_service'
import ModalService   from '@/shared/services/modal_service'

import { listenForTranslations, listenForReactions } from '@/shared/helpers/listen'

export default
  props:
    event: Object
    eventable: Object
  data: ->
    isEditCommentModalOpen: false
    isDeleteCommentModalOpen: false
    confirmOpts:
      submit: @eventable.destroy
      text:
        title:    'delete_comment_dialog.title'
        helptext: 'delete_comment_dialog.question'
        confirm:  'delete_comment_dialog.confirm'
        flash:    'comment_form.messages.destroyed'
  methods:
    openEditCommentModal: ->
      @isEditCommentModalOpen = true

    closeEditCommentModal: ->
      @isEditCommentModalOpen = false

    openDeleteCommentModal: ->
      @isDeleteCommentModalOpen = true

    closeDeleteCommentModal: ->
      @isDeleteCommentModalOpen = false
  created: ->
    @actions = [
      name: 'react'
      canPerform: => AbilityService.canAddComment(@eventable.discussion())
    ,
      name: 'reply_to_comment'
      icon: 'mdi-reply'
      canPerform: => AbilityService.canRespondToComment(@eventable)
      perform:    =>
        @$emit('reply-button-clicked', {event: @event, eventable: @eventable})
    ,
      name: 'edit_comment'
      icon: 'mdi-pencil'
      canPerform: => AbilityService.canEditComment(@eventable)
      perform:    => @openEditCommentModal()
    ,
      name: 'fork_comment'
      icon: 'mdi-call-split'
      canPerform: => AbilityService.canForkComment(@eventable)
      perform:    =>
        EventBus.broadcast $rootScope, 'toggleSidebar', false
        @event.toggleFromFork()
    ,
      name: 'translate_comment'
      icon: 'mdi-translate'
      canPerform: => @eventable.body && AbilityService.canTranslate(@eventable) && !@translation
      perform:    => @eventable.translate(Session.user().locale)
    ,
    #   name: 'copy_url'
    #   icon: 'mdi-link'
    #   canPerform: => clipboard.supported
    #   perform:    =>
    #     clipboard.copyText(LmoUrlService.event(@event, {}, absolute: true))
    #     FlashService.success("action_dock.comment_copied")
    # ,
      name: 'show_history'
      icon: 'mdi-history'
      canPerform: => @eventable.edited()
      perform:    => ModalService.open 'RevisionHistoryModal', model: => @eventable
    ,
      name: 'delete_comment'
      icon: 'mdi-delete'
      canPerform: => AbilityService.canDeleteComment(@eventable)
      perform:    => @openDeleteCommentModal()
    ]
  # mounted: ->
  #   listenForReactions($scope, $scope.eventable)
  #   listenForTranslations($scope)

</script>

<template lang="pug">
  .new-comment(id="'comment-'+ eventable.id")
    div(v-if='!eventable.translation')
    .thread-item__body.new-comment__body.lmo-markdown-wrapper(v-if='eventable.bodyFormat == "md"', v-marked='eventable.cookedBody()')
    .thread-item__body.new-comment__body.lmo-markdown-wrapper(v-if='eventable.bodyFormat == "html"', v-html='eventable.body')
    translation.thread-item__body(v-if='eventable.translation', :model='eventable', field='body')
    //- <outlet name="after-comment-body" model="eventable"></outlet>
    document-list(:model='eventable', :skip-fetch='true')
    attachment-list(:attachments="eventable.attachments")
    .lmo-md-actions
      //- <reactions_display model="eventable"></reactions_display>
      action-dock(:model='eventable', :actions='actions')
      v-dialog(v-model="isEditCommentModalOpen", lazy persistent)
        edit-comment-form(:comment="eventable", :close="closeEditCommentModal")
      v-dialog(v-model="isDeleteCommentModalOpen", lazy persistent)
        confirm-modal(:confirm="confirmOpts", :close="closeDeleteCommentModal")
    //- <outlet name="after-comment-event" model="eventable"></outlet>
</template>
