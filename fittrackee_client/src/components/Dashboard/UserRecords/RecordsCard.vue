<template>
  <div class="records-card">
    <Card>
      <template #title>
        <SportImage :sport-label="records.label" :color="records.color" />
        {{ sportTranslatedLabel }}
      </template>
      <template #content>
        <div
          class="record"
          v-for="record in getTranslatedRecords(records.records)"
          :key="record.id"
        >
          <span class="record-type">
            {{ record.label }}
          </span>
          <span class="record-value">{{ record.value }}</span>
          <span class="record-date">
            <router-link
              :to="{
                name: 'Workout',
                params: { workoutId: record.workout_id },
              }"
            >
              {{ record.workout_date }}
            </router-link>
          </span>
        </div>
      </template>
    </Card>
  </div>
</template>

<script setup lang="ts">
  import { toRefs } from 'vue'
  import { useI18n } from 'vue-i18n'

  import { ICardRecord, IRecord, IRecordsBySports } from '@/types/workouts'
  import { sortRecords } from '@/utils/records'

  interface Props {
    records: IRecordsBySports
    sportTranslatedLabel: string
  }
  const props = defineProps<Props>()

  const { records, sportTranslatedLabel } = toRefs(props)

  const { t } = useI18n()

  function getTranslatedRecords(records: IRecord[]): ICardRecord[] {
    const translatedRecords: ICardRecord[] = []
    records.map((record) => {
      translatedRecords.push({
        ...record,
        label: t(`workouts.RECORD_${record.record_type}`),
      })
    })
    return translatedRecords.sort(sortRecords)
  }
</script>

<style lang="scss" scoped>
  @import '~@/scss/vars.scss';

  .records-card {
    width: 100%;
    padding-bottom: $default-padding * 0.3;

    ::v-deep(.card) {
      font-size: 0.9em;
      .card-title {
        display: flex;
        font-size: 0.9em;
        .sport-img {
          padding-right: $default-padding;
          height: 20px;
          width: 20px;
        }
      }
      .card-content {
        font-size: 0.9em;
        padding: $default-padding;
        .record {
          display: flex;
          align-items: center;
          justify-content: space-between;
          span {
            padding: 2px;
          }
          .record-type {
            flex-grow: 1;
          }
          .record-value {
            font-weight: bold;
            white-space: nowrap;
            padding-right: $default-padding;
          }
          .record-date {
            white-space: nowrap;
            min-width: 30%;
            text-align: right;
          }
        }
      }
      @media screen and (max-width: $medium-limit) {
        font-size: 1em;
        .card-title {
          font-size: 1em;
          .sport-img {
            height: 22px;
            width: 22px;
          }
        }
      }
    }
  }
</style>
