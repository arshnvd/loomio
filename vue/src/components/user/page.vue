<style lang="scss">
.user-page { margin-top: 16px; }
.user-page__groups { margin-top: 24px; }
.user-page__groups-title { margin: 0; }
.user-page__group { padding: 0 !important; }

.user-page__contact-user {
  margin: 8px 0;
  width: 100%;
}
</style>

<script lang="coffee">
import Records        from '@/shared/services/records'
import EventBus       from '@/shared/services/event_bus'
import AbilityService from '@/shared/services/ability_service'
import ModalService   from '@/shared/services/modal_service'
import fromNow        from '@/mixins/from_now'
import urlFor         from '@/mixins/url_for'

import { applyLoadingFunction } from '@/shared/helpers/apply'

import _isEmpty     from 'lodash/isEmpty'
import _sortBy     from 'lodash/sortBy'

export default
  mixins: [fromNow, urlFor]
  data: ->
    user: {}
    isMembershipsFetchingDone: false
    isModalOpen: false
  created: ->
    # applyLoadingFunction(@, 'loadGroupsFor')
    @setUser()
    Records.users.findOrFetchById(@$route.params.key).then @setUser, (error) ->
      # EventBus.broadcast $rootScope, 'pageError', error
  methods:
    setUser: ->
      if @user = (Records.users.find(@$route.params.key) or Records.users.find(username: @$route.params.key))[0]
      # EventBus.broadcast $rootScope, 'currentComponent', {title: @user.name, page: 'userPage'}
        @loadGroupsFor(@user)

    openModal: ->
      @isModalOpen = true

    closeModal: ->
      @isModalOpen = false

    loadGroupsFor: (user) ->
      Records.memberships.fetchByUser(user)
  computed:
    canContactUser: ->
      @$store.getters.canContactUser(@user)

    isEmptyUser: ->
      _isEmpty @user

    orderedFormalGroups: ->
      _sortBy @user.formalGroups(), 'fullName'
</script>

<template lang="pug">
.loading-wrapper.container.main-container.lmo-one-column-layout
  loading(v-if='isEmptyUser')
  main.user-page.main-container.lmo-row(v-if='!isEmptyUser')
    .lmo-card.user-page__profile
      .user-page__content.lmo-flex(layout='row')
        .user-page__avatar.lmo-margin-right
          user-avatar(:user='user', size='featured')
          v-btn.md-block.md-primary.md-raised.user-page__contact-user(v-if='canContactUser', @click='openModal()', v-t="{ path: 'user_page.contact_user', args: { name: user.firstName() } }")
          v-dialog(v-model="isModalOpen", lazy, persistent)
            contact-request-modal(:user="user", :close="closeModal")
        .user-page__info
          h1.lmo-h1 {{user.name}}
          .lmo-hint-text @{{user.username}}
          p {{user.shortBio}}
          div(v-t="{ path: 'user_page.locale_field', args: { value: user.localeName() } }", v-if='user.localeName()')
          div(v-t="{ path: 'user_page.location_field', args: { value: user.location } }", v-if='user.location')
          div(v-t="{ path: 'user_page.online_field', args: { value: fromNow(user.lastSeenAt) } }", v-if='user.lastSeenAt')
          .user-page__groups
            h3.lmo-h3.user-page__groups-title(v-t="'common.groups'")
            v-list
              v-list-tile.user-page__group.lmo-flex.lmo-flex__center(v-for='group in user.formalGroups()', :key='group.id')
                img.md-avatar.lmo-box--small.lmo-margin-right(:src='group.logoUrl()')
                router-link(:to='urlFor(group)') {{group.fullName}}
            // <loading v-if="loadGroupsForExecuting"></loading>
</template>
