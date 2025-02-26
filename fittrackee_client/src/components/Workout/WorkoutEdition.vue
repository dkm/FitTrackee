<template>
  <div
    id="workout-edition"
    class="center-card with-margin"
    :class="{ 'center-form': workout && workout.with_gpx }"
  >
    <Card>
      <template #title>{{
        $t(`workouts.${isCreation ? 'ADD' : 'EDIT'}_WORKOUT`)
      }}</template>
      <template #content>
        <div id="workout-form">
          <form :class="{ errors: formErrors }" @submit.prevent="updateWorkout">
            <div class="form-items">
              <div class="form-item-radio" v-if="isCreation">
                <div>
                  <input
                    id="withGpx"
                    type="radio"
                    :checked="withGpx"
                    :disabled="loading"
                    @click="updateWithGpx"
                  />
                  <label for="withGpx">{{ $t('workouts.WITH_GPX') }}</label>
                </div>
                <div>
                  <input
                    id="withoutGpx"
                    type="radio"
                    :checked="!withGpx"
                    :disabled="loading"
                    @click="updateWithGpx"
                  />
                  <label for="withoutGpx">
                    {{ $t('workouts.WITHOUT_GPX') }}
                  </label>
                </div>
              </div>
              <div class="form-item">
                <label> {{ $t('workouts.SPORT', 1) }}*: </label>
                <select
                  id="sport"
                  required
                  @invalid="invalidateForm"
                  :disabled="loading"
                  v-model="workoutForm.sport_id"
                >
                  <option
                    v-for="sport in translatedSports"
                    :value="sport.id"
                    :key="sport.id"
                  >
                    {{ sport.translatedLabel }}
                  </option>
                </select>
              </div>
              <div class="form-item" v-if="isCreation && withGpx">
                <label for="gpxFile">
                  {{ $t('workouts.GPX_FILE') }}
                  {{ $t('workouts.ZIP_ARCHIVE_DESCRIPTION') }}*:
                </label>
                <input
                  id="gpxFile"
                  name="gpxFile"
                  type="file"
                  accept=".gpx, .zip"
                  :disabled="loading"
                  required
                  @invalid="invalidateForm"
                  @input="updateFile"
                />
                <div class="files-help info-box">
                  <div>
                    <strong>{{ $t('workouts.GPX_FILE') }}:</strong>
                    <ul>
                      <li>
                        {{ $t('workouts.MAX_SIZE') }}: {{ fileSizeLimit }}
                      </li>
                    </ul>
                  </div>
                  <div>
                    <strong>{{ $t('workouts.ZIP_ARCHIVE') }}:</strong>
                    <ul>
                      <li>{{ $t('workouts.NO_FOLDER') }}</li>
                      <li>
                        {{ $t('workouts.MAX_FILES') }}: {{ gpx_limit_import }}
                      </li>
                      <li>{{ $t('workouts.MAX_SIZE') }}: {{ zipSizeLimit }}</li>
                    </ul>
                  </div>
                </div>
              </div>
              <div class="form-item" v-else>
                <label for="title"> {{ $t('workouts.TITLE') }}: </label>
                <input
                  id="title"
                  name="title"
                  type="text"
                  :required="!isCreation"
                  @invalid="invalidateForm"
                  :disabled="loading"
                  v-model="workoutForm.title"
                />
              </div>
              <div v-if="!withGpx">
                <div class="workout-date-duration">
                  <div class="form-item">
                    <label>{{ $t('workouts.WORKOUT_DATE') }}*:</label>
                    <div class="workout-date-time">
                      <input
                        id="workout-date"
                        name="workout-date"
                        type="date"
                        required
                        @invalid="invalidateForm"
                        :disabled="loading"
                        v-model="workoutForm.workoutDate"
                      />
                      <input
                        id="workout-time"
                        name="workout-time"
                        class="workout-time"
                        type="time"
                        required
                        @invalid="invalidateForm"
                        :disabled="loading"
                        v-model="workoutForm.workoutTime"
                      />
                    </div>
                  </div>
                  <div class="form-item">
                    <label>{{ $t('workouts.DURATION') }}*:</label>
                    <div>
                      <input
                        id="workout-duration-hour"
                        name="workout-duration-hour"
                        class="workout-duration"
                        :class="{ errored: isDurationInvalid() }"
                        type="text"
                        placeholder="HH"
                        minlength="1"
                        maxlength="2"
                        pattern="^([0-1]?[0-9]|2[0-3])$"
                        required
                        @invalid="invalidateForm"
                        :disabled="loading"
                        v-model="workoutForm.workoutDurationHour"
                      />
                      :
                      <input
                        id="workout-duration-minutes"
                        name="workout-duration-minutes"
                        class="workout-duration"
                        :class="{ errored: isDurationInvalid() }"
                        type="text"
                        pattern="^([0-5][0-9])$"
                        minlength="2"
                        maxlength="2"
                        placeholder="MM"
                        required
                        @invalid="invalidateForm"
                        :disabled="loading"
                        v-model="workoutForm.workoutDurationMinutes"
                      />
                      :
                      <input
                        id="workout-duration-seconds"
                        name="workout-duration-seconds"
                        class="workout-duration"
                        :class="{ errored: isDurationInvalid() }"
                        type="text"
                        pattern="^([0-5][0-9])$"
                        minlength="2"
                        maxlength="2"
                        placeholder="SS"
                        required
                        @invalid="invalidateForm"
                        :disabled="loading"
                        v-model="workoutForm.workoutDurationSeconds"
                      />
                    </div>
                  </div>
                </div>
                <div class="workout-data">
                  <div class="form-item">
                    <label>
                      {{ $t('workouts.DISTANCE') }} ({{
                        authUser.imperial_units ? 'mi' : 'km'
                      }})*:
                    </label>
                    <input
                      :class="{ errored: isDistanceInvalid() }"
                      name="workout-distance"
                      type="number"
                      min="0"
                      step="0.001"
                      required
                      @invalid="invalidateForm"
                      :disabled="loading"
                      v-model="workoutForm.workoutDistance"
                    />
                  </div>
                  <div class="form-item">
                    <label>
                      {{ $t('workouts.ASCENT') }} ({{
                        authUser.imperial_units ? 'ft' : 'm'
                      }}):
                    </label>
                    <input
                      :class="{ errored: isElevationInvalid() }"
                      name="workout-ascent"
                      type="number"
                      min="0"
                      step="0.01"
                      @invalid="invalidateForm"
                      :disabled="loading"
                      v-model="workoutForm.workoutAscent"
                    />
                  </div>
                  <div class="form-item">
                    <label>
                      {{ $t('workouts.DESCENT') }} ({{
                        authUser.imperial_units ? 'ft' : 'm'
                      }}):
                    </label>
                    <input
                      :class="{ errored: isElevationInvalid() }"
                      name="workout-descent"
                      type="number"
                      min="0"
                      step="0.01"
                      @invalid="invalidateForm"
                      :disabled="loading"
                      v-model="workoutForm.workoutDescent"
                    />
                  </div>
                </div>
              </div>
              <div class="form-item">
                <label> {{ $t('workouts.NOTES') }}: </label>
                <CustomTextArea
                  name="notes"
                  :input="workoutForm.notes"
                  :disabled="loading"
                  @updateValue="updateNotes"
                />
              </div>
            </div>
            <ErrorMessage :message="errorMessages" v-if="errorMessages" />
            <div v-if="loading">
              <Loader />
            </div>
            <div v-else class="form-buttons">
              <button class="confirm" type="submit" :disabled="loading">
                {{ $t('buttons.SUBMIT') }}
              </button>
              <button class="cancel" @click.prevent="onCancel">
                {{ $t('buttons.CANCEL') }}
              </button>
            </div>
          </form>
        </div>
      </template>
    </Card>
  </div>
</template>

<script setup lang="ts">
  import {
    ComputedRef,
    Ref,
    computed,
    reactive,
    ref,
    toRefs,
    watch,
    onMounted,
    onUnmounted,
    withDefaults,
  } from 'vue'
  import { useI18n } from 'vue-i18n'
  import { useRouter } from 'vue-router'

  import { ROOT_STORE, WORKOUTS_STORE } from '@/store/constants'
  import { TAppConfig } from '@/types/application'
  import { ISport } from '@/types/sports'
  import { IAuthUserProfile } from '@/types/user'
  import { IWorkout, IWorkoutForm } from '@/types/workouts'
  import { useStore } from '@/use/useStore'
  import { formatWorkoutDate, getDateWithTZ } from '@/utils/dates'
  import { getReadableFileSize } from '@/utils/files'
  import { translateSports } from '@/utils/sports'
  import { convertDistance } from '@/utils/units'

  interface Props {
    authUser: IAuthUserProfile
    sports: ISport[]
    isCreation?: boolean
    loading?: boolean
    workout?: IWorkout
  }
  const props = withDefaults(defineProps<Props>(), {
    isCreation: false,
    loading: false,
    workout: () => ({} as IWorkout),
  })

  const { t } = useI18n()
  const store = useStore()
  const router = useRouter()

  const { authUser, workout, isCreation, loading } = toRefs(props)
  const translatedSports: ComputedRef<ISport[]> = computed(() =>
    translateSports(
      props.sports,
      t,
      'is_active_for_user',
      workout.value.id ? [workout.value.sport_id] : []
    )
  )
  const appConfig: ComputedRef<TAppConfig> = computed(
    () => store.getters[ROOT_STORE.GETTERS.APP_CONFIG]
  )
  const fileSizeLimit = appConfig.value.max_single_file_size
    ? getReadableFileSize(appConfig.value.max_single_file_size)
    : ''
  const gpx_limit_import = appConfig.value.gpx_limit_import
  const zipSizeLimit = appConfig.value.max_zip_file_size
    ? getReadableFileSize(appConfig.value.max_zip_file_size)
    : ''
  const errorMessages: ComputedRef<string | string[] | null> = computed(
    () => store.getters[ROOT_STORE.GETTERS.ERROR_MESSAGES]
  )
  const workoutForm = reactive({
    sport_id: '',
    title: '',
    notes: '',
    workoutDate: '',
    workoutTime: '',
    workoutDurationHour: '',
    workoutDurationMinutes: '',
    workoutDurationSeconds: '',
    workoutDistance: '',
    workoutAscent: '',
    workoutDescent: '',
  })
  const withGpx = ref(
    props.workout.id ? props.workout.with_gpx : props.isCreation
  )
  let gpxFile: File | null = null
  const formErrors = ref(false)
  const payloadErrorMessages: Ref<string[]> = ref([])

  onMounted(() => {
    if (props.workout.id) {
      formatWorkoutForm(props.workout)
    }
  })

  function updateNotes(value: string) {
    workoutForm.notes = value
  }
  function updateWithGpx() {
    withGpx.value = !withGpx.value
    formErrors.value = false
  }
  function updateFile(event: Event & { target: HTMLInputElement }) {
    if (event.target.files) {
      gpxFile = event.target.files[0]
    }
  }
  function formatWorkoutForm(workout: IWorkout) {
    workoutForm.sport_id = `${workout.sport_id}`
    workoutForm.title = workout.title
    workoutForm.notes = workout.notes
    if (!workout.with_gpx) {
      const workoutDateTime = formatWorkoutDate(
        getDateWithTZ(workout.workout_date, props.authUser.timezone),
        'yyyy-MM-dd'
      )
      const duration = workout.duration.split(':')
      workoutForm.workoutDistance = `${
        authUser.value.imperial_units
          ? convertDistance(workout.distance, 'km', 'mi', 3)
          : parseFloat(workout.distance.toFixed(3))
      }`
      workoutForm.workoutDate = workoutDateTime.workout_date
      workoutForm.workoutTime = workoutDateTime.workout_time
      workoutForm.workoutDurationHour = duration[0]
      workoutForm.workoutDurationMinutes = duration[1]
      workoutForm.workoutDurationSeconds = duration[2]
      workoutForm.workoutAscent =
        workout.ascent === null
          ? ''
          : `${
              authUser.value.imperial_units
                ? convertDistance(workout.ascent, 'm', 'ft', 2)
                : parseFloat(workout.ascent.toFixed(2))
            }`
      workoutForm.workoutDescent =
        workout.descent === null
          ? ''
          : `${
              authUser.value.imperial_units
                ? convertDistance(workout.descent, 'm', 'ft', 2)
                : parseFloat(workout.descent.toFixed(2))
            }`
    }
  }
  function isDistanceInvalid() {
    return payloadErrorMessages.value.includes('workouts.INVALID_DISTANCE')
  }
  function isDurationInvalid() {
    return payloadErrorMessages.value.includes('workouts.INVALID_DURATION')
  }
  function isElevationInvalid() {
    return payloadErrorMessages.value.includes(
      'workouts.INVALID_ASCENT_OR_DESCENT'
    )
  }
  function formatPayload(payload: IWorkoutForm) {
    payloadErrorMessages.value = []
    payload.title = workoutForm.title
    payload.duration =
      +workoutForm.workoutDurationHour * 3600 +
      +workoutForm.workoutDurationMinutes * 60 +
      +workoutForm.workoutDurationSeconds
    if (payload.duration <= 0) {
      payloadErrorMessages.value.push('workouts.INVALID_DURATION')
    }
    payload.distance = authUser.value.imperial_units
      ? convertDistance(+workoutForm.workoutDistance, 'mi', 'km', 3)
      : +workoutForm.workoutDistance
    if (payload.distance <= 0) {
      payloadErrorMessages.value.push('workouts.INVALID_DISTANCE')
    }
    payload.workout_date = `${workoutForm.workoutDate} ${workoutForm.workoutTime}`
    payload.ascent =
      workoutForm.workoutAscent === ''
        ? null
        : authUser.value.imperial_units
        ? convertDistance(+workoutForm.workoutAscent, 'ft', 'm', 3)
        : +workoutForm.workoutAscent
    payload.descent =
      workoutForm.workoutDescent === ''
        ? null
        : authUser.value.imperial_units
        ? convertDistance(+workoutForm.workoutDescent, 'ft', 'm', 3)
        : +workoutForm.workoutDescent
    if (
      (payload.ascent !== null && payload.descent === null) ||
      (payload.ascent === null && payload.descent !== null)
    ) {
      payloadErrorMessages.value.push('workouts.INVALID_ASCENT_OR_DESCENT')
    }
  }
  function updateWorkout() {
    const payload: IWorkoutForm = {
      sport_id: +workoutForm.sport_id,
      notes: workoutForm.notes,
    }
    if (props.workout.id) {
      if (props.workout.with_gpx) {
        payload.title = workoutForm.title
      } else {
        formatPayload(payload)
      }
      if (payloadErrorMessages.value.length > 0) {
        store.commit(
          ROOT_STORE.MUTATIONS.SET_ERROR_MESSAGES,
          payloadErrorMessages.value
        )
      } else {
        store.dispatch(WORKOUTS_STORE.ACTIONS.EDIT_WORKOUT, {
          workoutId: props.workout.id,
          data: payload,
        })
      }
    } else {
      if (withGpx.value) {
        if (!gpxFile) {
          const errorMessage = 'workouts.NO_FILE_PROVIDED'
          store.commit(ROOT_STORE.MUTATIONS.SET_ERROR_MESSAGES, errorMessage)
          return
        }
        payload.file = gpxFile
        store.dispatch(WORKOUTS_STORE.ACTIONS.ADD_WORKOUT, payload)
      } else {
        formatPayload(payload)
        if (payloadErrorMessages.value.length > 0) {
          store.commit(
            ROOT_STORE.MUTATIONS.SET_ERROR_MESSAGES,
            payloadErrorMessages.value
          )
        } else {
          store.dispatch(
            WORKOUTS_STORE.ACTIONS.ADD_WORKOUT_WITHOUT_GPX,
            payload
          )
        }
      }
    }
  }
  function onCancel() {
    if (props.workout.id) {
      router.push({
        name: 'Workout',
        params: { workoutId: props.workout.id },
      })
    } else {
      router.go(-1)
    }
  }
  function invalidateForm() {
    formErrors.value = true
  }

  onUnmounted(() => store.commit(ROOT_STORE.MUTATIONS.EMPTY_ERROR_MESSAGES))

  watch(
    () => props.workout,
    async (
      newWorkout: IWorkout | undefined,
      previousWorkout: IWorkout | undefined
    ) => {
      if (newWorkout !== previousWorkout && newWorkout && newWorkout.id) {
        formatWorkoutForm(newWorkout)
      }
    }
  )
</script>

<style lang="scss" scoped>
  @import '~@/scss/vars.scss';

  #workout-edition {
    ::v-deep(.card) {
      .card-title {
        text-align: center;
        text-transform: uppercase;
      }

      .card-content {
        @media screen and (max-width: $medium-limit) {
          padding: $default-padding 0;
        }

        #workout-form {
          .form-items {
            display: flex;
            flex-direction: column;

            input {
              height: 20px;
            }

            .workout-date-duration {
              display: flex;
              flex-direction: row;
              justify-content: space-between;

              @media screen and (max-width: $medium-limit) {
                flex-direction: column;
              }
            }

            .form-item {
              display: flex;
              flex-direction: column;
              padding: $default-padding;

              .workout-date-time {
                display: flex;
                #workout-date {
                  margin-right: $default-margin;
                }
              }

              .workout-duration {
                width: 25px;
              }
            }

            .form-item-radio {
              display: flex;
              justify-content: space-around;
              label {
                font-weight: normal;
                @media screen and (max-width: $medium-limit) {
                  font-size: 0.9em;
                }
              }
              input {
                margin-top: -2px;
                vertical-align: middle;
              }
            }
          }

          .form-buttons {
            display: flex;
            justify-content: flex-end;
            button {
              margin: $default-padding * 0.5;
            }
          }

          .files-help {
            display: flex;
            justify-content: space-around;
            margin-top: $default-margin;
            div {
              display: flex;
              @media screen and (max-width: $medium-limit) {
                flex-direction: column;
              }
              ul {
                margin: 0;
                padding: 0 $default-padding * 2;
              }
            }
          }

          .workout-data {
            display: flex;
            flex-direction: row;
            justify-content: space-between;
            .form-item {
              width: 30%;
            }

            @media screen and (max-width: $medium-limit) {
              flex-direction: column;
              .form-item {
                width: initial;
              }
            }
          }
        }
      }
    }

    @media screen and (max-width: $small-limit) {
      margin-bottom: 0;
      &.center-form {
        margin: 50px auto;
      }

      &.with-margin {
        margin-top: 0;
      }
    }

    .errored {
      outline: 2px solid var(--input-error-color);
    }
  }
</style>
