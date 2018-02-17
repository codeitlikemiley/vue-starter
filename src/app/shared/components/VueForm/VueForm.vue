<template>
  <vue-panel>
    <vue-panel-header v-if="schema.title"
                      :title="schema.title"
                      :subtitle="schema.subtitle" />
    <vue-panel-body>
      <form :class="$style.vueForm" :name="schema.name" :id="schema.id" @submit.prevent="submitForm()">

        <div v-for="element in elements">
          <keep-alive>
            <component
              :key="element.model"
              :is="element.type"
              v-bind="getComponentProps(element)"
              v-on="getComponentHandler(element)" />
          </keep-alive>
        </div>

        <vue-button
          primary
          :disabled="submitDisabled()"
          v-if="schema.submitText">
          {{ schema.submitText }}
        </vue-button>

        <vue-button
          v-if="schema.cancellationText"
          @click.prevent="reset(true)">
          {{ schema.cancellationText }}
        </vue-button>
      </form>
    </vue-panel-body>
  </vue-panel>
</template>

<script lang="ts">
  import VuePanel         from '../VuePanel/VuePanel.vue';
  import VuePanelHeader   from '../VuePanel/VuePanelHeader/VuePanelHeader.vue';
  import VuePanelBody     from '../VuePanel/VuePanelBody/VuePanelBody.vue';
  import VueInput         from '../VueInput/VueInput.vue';
  import VueCheckBox      from '../VueCheckBox/VueCheckBox.vue';
  import VueButton        from '../VueButton/VueButton.vue';
  import { IFormElement } from './IFormSchema';

  export default {
    name:       'VueForm',
    components: {
      VuePanel,
      VuePanelHeader,
      VuePanelBody,
      VueInput,
      VueCheckBox,
      VueButton,
    },
    props:      {
      schema: {
        type:     Object,
        required: true,
      },
    },
    data(): any {
      return {
        elements: [],
      };
    },
    computed:   {},
    methods:    {
      submitDisabled(): boolean {
        let submitDisabled: boolean = false;

        this.$data.elements.forEach((element: IFormElement) => {
          if (this.isValid(element) === false) {
            submitDisabled = true;
          }

          if (element.required && !element.value) {
            submitDisabled = true;
          }
        });

        return submitDisabled;
      },
      submitForm(): void {
        const model: any = {};

        this.$data.elements.forEach((element: IFormElement) => {
          model[element.model] = element.value;
        });

        if (this.submitDisabled() === false) {
          this.$emit('submit', model, this.reset);
        }
      },
      isValid(element: IFormElement): boolean {
        if (!element.value || !element.isValid) {
          return true;
        }

        return element.isValid(element.value);
      },
      getComponentProps(element: IFormElement): any {
        const isValid: boolean = this.isValid(element);
        if (element.type === 'vue-input') {
          return {
            placeholder: element.label || '',
            required:    element.required || false,
            type:        element.inputType || 'text',
            value:       element.value,
            name:        element.model,
            isValid:     isValid,
            message:     isValid ? '' : element.invalidText,
          };
        } else {
          return {
            label:   element.label || '',
            checked: element.value,
            name:    element.model,
          };
        }
      },
      getComponentHandler(element: IFormElement): any {
        if (element.type === 'vue-input') {
          return {
            change: (e: any) => {
              element.value = e.target.value;
              this.$forceUpdate();
            },
          };
        } else {
          return {
            click: (e: any) => {
              element.value = !element.value;
              this.$forceUpdate();
            },
          };
        }
      },
      reset(emit: boolean = false) {
        const elements: IFormElement[] = JSON.parse(JSON.stringify(this.schema.elements));
        const isValid = () => {
          return true;
        };

        this.$data.elements = elements.map((element: IFormElement, idx: number) => {
          element.value = element.value || null;
          element.isValid = this.schema.elements[idx].isValid ? this.schema.elements[idx].isValid : isValid;

          return element;
        });

        if (emit) {
          this.$emit('reset');
        }
      },
    },
    created() {
      this.reset();
    },
  };
</script>

<style lang="scss" module>
  @import "../../styles";

  .vueForm {
    display: block;

    button {
      margin-top:    $grid-unit * 5;
      margin-bottom: 0;
    }
  }
</style>