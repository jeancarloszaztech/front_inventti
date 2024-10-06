<template>
  <q-page class="flex-center">
    <div class="row">
      <div class="col-2"></div>
      <div class="col">
        <q-input filled v-model="textSearchMod" :label="$t('pages_General_TextSearchFieldDesc')"
          @update:model-value="onTextSearchChange" maxlength="12">
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </div>
      <div class="col-2"></div>
    </div>
    <div class="row">
      <div class="col-2"></div>
      <div class="q-pa-md q-gutter-md col">
        <q-list bordered class="rounded-borders">
          <q-item-label header>{{ $t("pages_General_TotalCountDesc") }}
            {{ itemsCountMod }}</q-item-label>
          <div ref="scrollTargetRef" class="q-pa-md" style="max-height: 60vh; overflow: auto">
            <q-infinite-scroll @load="onScroll" :offset="offsetRef" :scroll-target="scrollTargetRef">
              <q-item v-for="(item, index) in itemsMod" :key="index">
                <q-item-section>
                  <q-item-label>{{ item.summary }}</q-item-label>
                  <q-item-label caption lines="2">{{ item.momment }} Temp:
                    {{ item.temperatureC }}ºC</q-item-label>
                </q-item-section>

                <q-item-section top side>
                  <div class="text-grey-8 q-gutter-xs">
                    <q-btn class="gt-xs" size="12px" flat dense round icon="edit" @click="formOpened = true" />
                  </div>
                </q-item-section>
              </q-item>
            </q-infinite-scroll>
          </div>
        </q-list>

        <q-page-sticky position="bottom-right" :offset="[48, 48]">
          <q-btn fab icon="add" color="primary" @click="cleanFormFields" />
        </q-page-sticky>
      </div>
      <q-dialog v-model="formOpenedMod">
        <q-card style="min-width: 350px">
          <q-card-section>
            <div class="text-h6">Consulta de clima</div>
          </q-card-section>

          <q-card-section class="q-pt-none">
            <q-input v-model="latitudeRef" label="Latitude" @keyup.enter="formOpenedMod = false" />
            <q-input v-model="longitudeRef" label="Longitude" @keyup.enter="formOpenedMod = false" />
          </q-card-section>

          <q-card-actions align="right" class="text-primary">
            <q-btn flat label="Cancel" v-close-popup />
            <q-btn flat label="Consultar" v-close-popup @click="consultarClima" />
          </q-card-actions>
        </q-card>
      </q-dialog>
      <div class="col-2"></div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import { api } from "boot/axios";
import { useQuasar } from "quasar";
import { useI18n } from "vue-i18n";

export default defineComponent({
  name: "ConsultasClimaPage",
  setup() {
    const $t = useI18n().t;
    const $q = useQuasar();

    const textSearchMod = ref(null);
    const itemsCountMod = ref(0);
    const itemsMod = ref([]);
    const scrollTargetRef = ref(null);
    let offsetRef = ref(0);
    const latitudeRef = ref("");
    const longitudeRef = ref("");
    const formOpenedMod = ref(false);

    //DD/MM HH:mm
    let formatDate = (momment) => {
      let mommentOption = new Date(momment);
      //DD/MM HH:mm
      let day = String(mommentOption.getDate()).padStart(2, "0");
      let month = String(mommentOption.getMonth() + 1).padStart(2, "0");
      let hours = String(mommentOption.getHours()).padStart(2, "0");
      let minutes = String(mommentOption.getMinutes()).padStart(2, "0");
      let formattedDate = `${day}/${month} ${hours}:${minutes}`;

      return formattedDate;
    }

    let resetItemsFn = () => {
      itemsCountMod.value = 1;
      itemsMod.value = [];
    };

    let onLoadItemsFn = (index, doneFn) => {
      let url = `api/v1/clima-tempo?paging-size=20&paging-page=0`;

      if (textSearchMod.value && textSearchMod.value.length > 3) {
        url += `&text-search=${textSearchMod.value}`;
      }

      api
        .get(url)
        .then((response) => {
          itemsCountMod.value = response.data.totalizers.totalCount;
          let options = response.data.data;
          //If do tamanho removido para deixar a consulta em branco quando não tiver elementos
          //Foi resolvido um bug pois estava concatenando cada resposta da API quando o usuário colocava mais texto para procura

          //1. Aqui foi criada a lógica para formatação da data de acordo com o que foi especificado
          let result = [];
          for (const option of options) {
            let formattedDate = formatDate(option.momment);

            result.push({ ...option, momment: formattedDate });
          }

          itemsMod.value = result;
          offsetRef.value++;
        })
        .catch((error) => {
          $q.notify({
            color: "negative",
            position: "top",
            message: $t("pages_General_LoadingFailedMsg"),
            icon: "report_problem",
          });
        })
        .finally(() => {
          offsetRef.value = 0;
        });
    };

    // carga inicial
    setTimeout(() => onLoadItemsFn(), 50);

    let onScroll = (index, done) => {
      if (itemsMod.value.length >= itemsCountMod.value) {
        done(); // Finaliza o scroll sem fazer requisição
        return;
      }

      console.log(itemsMod.value.length);
      console.log(itemsCountMod.value);

      let url = `api/v1/clima-tempo?paging-size=20&paging-page=${offsetRef.value}`;

      if (textSearchMod.value && textSearchMod.value.length > 3) {
        url += `&text-search=${textSearchMod.value}`;
      }

      api
        .get(url)
        .then((response) => {
          const responseData = response.data.data;
          let result = [];
          for (const option of responseData) {
            let formattedDate = formatDate(option.momment);

            result.push({ ...option, momment: formattedDate });
          }

          itemsMod.value = [...itemsMod.value, ...result];
          offsetRef.value++;
        })
        .catch((error) => {
          console.log(error);
          $q.notify({
            color: "negative",
            position: "top",
            message: $t("pages_General_LoadingFailedMsg"),
            icon: "report_problem",
          });
        })
        .finally(() => done());
    };

    let cleanFormFields = () => {
      latitudeRef.value = "";
      longitudeRef.value = "";
      formOpenedMod.value = true;
    }

    let consultarClima = () => {
      let url = `api/v1/clima-tempo/sincrono?lat=${latitudeRef.value}&lon=${longitudeRef.value}`;

      api
        .get(url)
        .then((response) => {
          const responseData = response.data.data;
          resetItemsFn();

          let formattedDate = formatDate(responseData.momment);

          itemsMod.value.push({ ...responseData, momment: formattedDate });
        })
        .catch((error) => {
          console.log(error);
          $q.notify({
            color: "negative",
            position: "top",
            message: $t("pages_General_LoadingFailedMsg"),
            icon: "report_problem",
          });
        });
    }

    return {
      formOpenedMod,
      formUsernameMod: ref(null),
      formIsAdminMod: ref(false),
      textSearchMod,
      itemsCountMod,
      itemsMod,
      scrollTargetRef,
      offsetRef,
      latitudeRef,
      longitudeRef,

      onScroll,
      cleanFormFields,
      consultarClima,
      onLoadItems: onLoadItemsFn,
      onTextSearchChange(inputValue, doneFn, abortFn) {
        onLoadItemsFn();
      },
    };
  },
});
</script>