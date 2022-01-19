<template>
  <div>
    <ListQuotes
      :quotes="quotes"
      :listen-quotes="listenQuotes"
      @unlisten="onUnlisten"
    />
    <div class="mt-2 text-right">
      <cite class="text-small">
        Atualizará novamente em <b>{{ nextUpdateTime }} segundos</b></cite
      >
    </div>
  </div>
</template>

<script>
import { ref, reactive, toRefs } from '@vue/reactivity'
import ListQuotes from './ListQuotes.vue'
import api from '../services/api'
import { onMounted, onUnmounted, watch } from '@vue/runtime-core'

export default {
  components: { ListQuotes },
  props: {
    listenQuotes: { type: Array, required: true },
  },
  emits: ['unlisten'],
  setup(props, { emit }) {
    const quotes = ref({})
    const nextUpdateTime = ref(30)
    const interval = ref(null)

    const methods = reactive({
      async refresh() {
        const { data } = await api.listen(props.listenQuotes)
        quotes.value = data
      },
      onUnlisten(code) {
        emit('unlisten', code)
      },
      restartInterval() {
        clearInterval(interval.value)
        nextUpdateTime.value = 30
        interval.value = setInterval(() => {
          nextUpdateTime.value -= 1
          if (nextUpdateTime.value == 0) {
            nextUpdateTime.value = 30
            this.refresh()
          }
        }, 1000)
      },
    })

    onUnmounted(() => {
      clearInterval(interval.value)
    })

    onMounted(() => {
      methods.refresh()
      methods.restartInterval()
    })

    watch(props, () => {
      methods.refresh()
      methods.restartInterval()
    })

    return { quotes, nextUpdateTime, interval, ...toRefs(methods) }
  },
}
</script>

<style lang="scss" scoped>
</style>