<template>
  <AposInputWrapper
    :field="field"
    :error="effectiveError"
    :uid="uid"
    :modifiers="modifiers"
    :display-options="displayOptions"
  >
    <template #body>
      <AposSelect
        :icon="icon"
        :choices="choices"
        :disabled="field.readOnly"
        :selected="value.data"
        @change="change"
      />
    </template>
  </AposInputWrapper>
</template>

<script>
import AposInputMixin from 'Modules/@apostrophecms/schema/mixins/AposInputMixin';

export default {
  name: 'AposInputSelect',
  mixins: [ AposInputMixin ],
  props: {
    icon: {
      type: String,
      default: 'menu-down-icon'
    }
  },
  data() {
    return {
      next: (this.value.data == null) ? null : this.value.data,
      choices: []
    };
  },
  async mounted() {
    let choices;
    if (typeof this.field.choices === 'string') {
      const action = this.options.action;
      const response = await apos.http.get(
        `${action}/choices`,
        {
          qs: {
            fieldId: this.field._id
          },
          busy: true
        }
      );
      if (response.choices) {
        choices = response.choices;
      }
    } else {
      choices = this.field.choices;
    }
    // Add an null option if there isn't one already
    if (!this.field.required && !choices.find(choice => {
      return choice.value === null;
    })) {
      this.choices.push({
        label: '',
        value: null
      });
    }
    this.choices = this.choices.concat(choices);
    this.$nextTick(() => {
      // this has to happen on nextTick to avoid emitting before schemaReady is
      // set in AposSchema
      if (this.field.required && (this.next == null) && (this.choices[0] != null)) {
        this.next = this.choices[0].value;
      }
    });
  },
  methods: {
    validate(value) {
      if (this.field.required && (value === null)) {
        return 'required';
      }

      if (value && !this.choices.find(choice => choice.value === value)) {
        return 'invalid';
      }

      return false;
    },
    change(value) {
      // Allows expression of non-string values
      this.next = this.choices.find(choice => choice.value === JSON.parse(value)).value;
    }
  }
};
</script>
