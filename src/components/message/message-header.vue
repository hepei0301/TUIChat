<template>
  <div class="base" :class="[ isMine ? 'right' : 'left']">
    <div class="name text-ellipsis">{{ from }}</div>
    <div class="date">{{ date }}</div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import { getFullDate } from '../../utils/date'
export default {
  name: 'MessageHeader',
  props: {
    message: {
      type: Object,
      required: true
    }
  },
  computed: {
    ...mapState({
      currentConversation: state => state.conversation.currentConversation,
      currentUserProfile: state => state.user.currentUserProfile,
      currentMemberList: state => state.group.currentMemberList
    }),
    date() {
      return getFullDate(new Date(this.message.time * 1000))
    },
    from() {
      const isC2C = this.currentConversation.type === this.TIM.TYPES.CONV_C2C
      // 自己发送的用昵称渲染
      if (this.isMine) {
        return this.message.nameCard || this.currentUserProfile.nick || this.currentUserProfile.userID
      }
      // 1. C2C 的消息体中还无 nick / avatar 字段，需从 conversation.userProfile 中获取
      if (isC2C) {
        return (
          this.currentConversation.userProfile.nick ||
          this.currentConversation.userProfile.userID
        )
      } else if (this.currentConversation.type === this.TIM.TYPES.CONV_SYSTEM) {
        return this.message.type === this.TIM.TYPES.MSG_GRP_SYS_NOTICE
          ? '群系统通知'
          : '系统通知'
      }
      // 2. 群组消息，用消息体中的 nick 渲染。nameCard暂时支持不完善
      return this.message.nameCard ||  this.message.nick || this.message.from
    },
    isMine() {
      return this.message.flow === 'out'
    }
  }
}
</script>

<style lang="stylus" scoped>
.right {
  display: flex;
  flex-direction: row-reverse;
}

.left {
  display: flex;
  flex-direction: row;
}

.base {
  color: $secondary;
  font-size: 12px;
}

.name {
  padding: 0 4px;
  max-width: 100px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>
