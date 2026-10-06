<q-page class="q-pa-md flex flex-center">
        <q-card class="modal-shell bg-grey-10 text-grey-2 shadow-24 column no-wrap">
          <q-card-section class="q-px-lg q-py-md row items-center no-wrap"
            style="border-bottom:1px solid rgba(255,255,255,.08)">
            <div class="col">
              <div class="text-h5 text-weight-bold">Скважина № {{ well.number }}</div>
              <div class="row q-gutter-md q-mt-xs text-caption text-grey-5">
                <span>{{ well.field }}</span><span>Глубина {{ well.depth }} м</span><span>Бурение {{ well.drillingDate
                  }}</span>
              </div>
            </div>
            <q-chip dense color="positive" text-color="white" icon="circle">Активная</q-chip>
            <q-btn flat round dense icon="close" class="q-ml-sm" @click="notify('Закрытие окна')" />
          </q-card-section>

          <div class="row col no-wrap overflow-hidden">
            <aside class="layers-panel q-pa-md column no-wrap" style="border-right:1px solid rgba(255,255,255,.08)">
              <div class="row items-center q-mb-sm">
                <div class="text-subtitle1 text-weight-medium col">Литологический разрез</div><q-btn flat dense round
                  icon="add" @click="addLayer" />
              </div>
              <q-input v-model="search" dense outlined dark placeholder="Поиск слоя" clearable class="q-mb-sm"><template
                  #prepend><q-icon name="search" /></template></q-input>
              <div class="layer-scroll col overflow-auto q-pr-xs">
                <q-list bordered separator class="rounded-borders">
                  <q-item v-for="layer in filteredLayers" :key="layer.id" clickable :active="layer.id === selectedId"
                    active-class="layer-item active" class="layer-item q-py-sm" @click="selectLayer(layer.id)">
                    <q-item-section avatar>
                      <div class="lith-dot"
                        :style="{ background: layer.color, width: '12px', height: '42px', borderRadius: '4px' }" />
                    </q-item-section>
                    <q-item-section><q-item-label class="text-weight-medium">{{ layer.kind
                        }}</q-item-label><q-item-label caption class="text-grey-5">{{ fmt(layer.top) }} — {{
                          fmt(layer.bottom) }} м</q-item-label></q-item-section>
                  </q-item>
                </q-list>
              </div>
              <q-btn outline color="primary" icon="add" label="Добавить слой" class="full-width q-mt-sm"
                @click="addLayer" />
            </aside>

            <main class="main-panel col column no-wrap">
              <div class="q-pa-md q-pb-sm row items-center no-wrap">
                <div class="col">
                  <div class="text-h6 text-weight-medium">Слой {{ selectedLayer.id }}</div>
                  <div class="text-caption text-grey-5">{{ selectedLayer.kind }} · {{ fmt(selectedLayer.top) }}—{{
                    fmt(selectedLayer.bottom) }} м</div>
                </div>
                <q-select v-model="selectedLayer.kind" :options="lithologyOptions" dense outlined dark emit-value
                  map-options style="width:180px" label="Литология" />
              </div>
              <q-tabs v-model="tab" dense no-caps align="left" active-color="primary" indicator-color="primary"
                class="q-px-md" style="border-bottom:1px solid rgba(255,255,255,.08)">
                <q-tab name="main" label="Основные" /><q-tab name="physical" label="Физические" /><q-tab name="extra"
                  label="Дополнительные" /><q-tab name="mechanical" label="Механические" /><q-tab name="fracture"
                  label="Трещиноватость" />
              </q-tabs>

              <div class="col overflow-auto q-pa-md">
                <template v-if="tab === 'main'">
                  <div class="section-grid">
                    <q-card flat bordered class="bg-grey-10 q-pa-md">
                      <div class="text-subtitle2 q-mb-sm">Глубина и геометрия</div>
                      <div class="property-grid">
                        <NumField v-model="selectedLayer.top" label="Кровля, м" @update:modelValue="syncPower" />
                        <NumField v-model="selectedLayer.bottom" label="Подошва, м" @update:modelValue="syncPower" />
                        <NumField v-model="selectedLayer.power" label="Мощность, м" readonly />
                        <NumField v-model="selectedLayer.id" label="№ слоя" readonly />
                      </div>
                    </q-card>
                    <q-card flat bordered class="bg-grey-10 q-pa-md">
                      <div class="text-subtitle2 q-mb-sm">Литологические характеристики</div>
                      <div class="property-grid">
                        <SelectField v-model="selectedLayer.kind" :options="lithologyOptions" label="Вид" />
                        <SelectField v-model="selectedLayer.composition"
                          :options="['Песчаный', 'Глинистый', 'Карбонатный', 'Смешанный']" label="Состав" />
                        <SelectField v-model="selectedLayer.color"
                          :options="['Светло-серый', 'Серый', 'Жёлтый', 'Коричневый', 'Красноватый']" label="Цвет" />
                        <SelectField v-model="selectedLayer.consistency"
                          :options="['Плотная', 'Пластичная', 'Твёрдая', 'Мягкая']" label="Консистенция" />
                        <SelectField v-model="selectedLayer.minComposition"
                          :options="['Кварцевый', 'Карбонат', 'Глинистый', 'Смешанный']" label="Мин. состав" />
                        <SelectField v-model="selectedLayer.decomposition"
                          :options="['Нет', 'Слабое', 'Среднее', 'Сильное']" label="Разложение" />
                        <SelectField v-model="selectedLayer.torf" :options="['Нет', 'Слабая', 'Средняя', 'Сильная']"
                          label="Заторфованность" />
                      </div>
                    </q-card>
                  </div>
                  <q-card flat bordered class="bg-grey-10 q-pa-md q-mt-md">
                    <div class="text-subtitle2 q-mb-sm">Включения</div>
                    <div class="property-grid">
                      <NumField v-model="selectedLayer.filler" label="Заполнитель, %" />
                      <NumField v-model="selectedLayer.interlayer" label="Прослойка, %" />
                      <NumField v-model="selectedLayer.lens" label="Линза, %" />
                    </div>
                  </q-card>
                </template>
                <PropertyGroup v-else-if="tab === 'physical'" title="Физические свойства" :fields="physicalFields"
                  v-model="selectedLayer" />
                <PropertyGroup v-else-if="tab === 'extra'" title="Дополнительные характеристики" :fields="extraFields"
                  v-model="selectedLayer" />
                <PropertyGroup v-else-if="tab === 'mechanical'" title="Механические свойства" :fields="mechanicalFields"
                  v-model="selectedLayer" />
                <PropertyGroup v-else title="Трещиноватость" :fields="fractureFields" v-model="selectedLayer" />
              </div>
              <div class="q-px-md q-py-sm row items-center" style="border-top:1px solid rgba(255,255,255,.08)">
                <div class="col text-caption text-grey-5">{{ selectedLayer.kind }} · {{ fmt(selectedLayer.top) }}—{{
                  fmt(selectedLayer.bottom) }} м · мощность {{ fmt(selectedLayer.power) }} м</div><q-btn flat
                  label="Отмена" @click="notify('Изменения отменены')" /><q-btn color="primary" unelevated
                  label="Сохранить" class="q-ml-sm" @click="save" />
              </div>
            </main>
          </div>
        </q-card>
      </q-page
