<template>
  <v-container class="fill-height">
    <v-row justify="center" align="center">
      <v-col cols="12" sm="10" md="8" lg="6">
        <v-card class="mx-auto pa-4">
          <v-card-title class="text-center text-h5 py-4">
            AI 상담 지원 신청서
          </v-card-title>

          <v-card-text>
            <v-form ref="form" v-model="isFormValid" @submit.prevent="submitForm">
              <!-- 기본 정보 섹션 -->
              <v-row>
                <v-col cols="12">
                  <v-text-field
                    v-model="formData.name"
                    label="이름"
                    :rules="[rules.required]"
                    outlined
                    dense
                    color="indigo-darken-2"
                    background-color="indigo-lighten-5"
                  ></v-text-field>
                </v-col>

                <v-col cols="12">
                  <v-select
                    v-model="formData.gender"
                    :items="genderOptions"
                    label="성별"
                    :rules="[rules.required]"
                    outlined
                    dense
                    color="indigo-darken-2"
                    background-color="indigo-lighten-5"
                  ></v-select>
                </v-col>

                <v-col cols="12">
                  <v-text-field
                    v-model="formData.age"
                    label="나이"
                    type="number"
                    :rules="[rules.required, rules.ageLimit]"
                    outlined
                    dense
                    color="indigo-darken-2"
                    background-color="indigo-lighten-5"
                  ></v-text-field>
                </v-col>
              </v-row>

              <!-- 지원 신청 사유 -->
              <v-textarea
                v-model="formData.reason"
                label="정책 지원신청 사유"
                :rules="[rules.required, rules.minLength]"
                counter
                outlined
                auto-grow
                rows="3"
                color="indigo-darken-2"
                background-color="indigo-lighten-5"
              ></v-textarea>

              <!-- 기대 효과 -->
              <v-textarea
                v-model="formData.expectation"
                label="정책 지원 참여를 통해 기대하는 효과"
                :rules="[rules.required, rules.minLength]"
                counter
                outlined
                auto-grow
                rows="3"
                color="indigo-darken-2"
                background-color="indigo-lighten-5"
              ></v-textarea>

              <!-- 기계 선택 -->
              <v-card class="mb-4 pa-4" outlined>
                <v-card-subtitle class="pb-0">기계 선택 (중복 가능)</v-card-subtitle>
                <v-checkbox
                  v-model="formData.machines"
                  v-for="machine in machineOptions"
                  :key="machine.value"
                  :label="machine.text"
                  :value="machine.value"
                  :rules="[rules.atLeastOne]"
                  color="indigo-darken-2"
                ></v-checkbox>
              </v-card>

              <!-- 전화번호 -->
              <v-text-field
                v-model="formData.phone"
                label="전화번호"
                :rules="[rules.required, rules.phoneFormat]"
                placeholder="010-0000-0000"
                outlined
                dense
                color="indigo-darken-2"
                background-color="indigo-lighten-5"
              ></v-text-field>

              <!-- 개인정보 동의 -->
              <v-checkbox
                v-model="formData.agreement"
                :rules="[rules.agreement]"
                class="mt-4"
                color="indigo-darken-2"
                multiple
              >
                <template v-slot:label>
                  <div>
                    개인정보 수집 및 이용에 동의합니다.
                    <span
                      @click.stop="showPrivacyDialog = true"
                      class="text-grey-lighten-1 text-decoration-underline cursor-pointer"
                      style="margin-left: 4px;"
                    >
                      자세히 보기
                    </span>
                  </div>
                </template>
              </v-checkbox>
            </v-form>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="indigo-darken-2"
              large
              :disabled="!isFormValid"
              @click="submitForm"
            >
              신청하기
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>

    <!-- 개인정보 처리방침 다이얼로그 -->
    <v-dialog v-model="showPrivacyDialog" max-width="600px">
      <v-card>
        <v-card-title>개인정보 처리방침</v-card-title>
        <v-card-text>
          <p>1. 수집하는 개인정보 항목</p>
          <p>- 이름, 성별, 나이, 전화번호</p>
          <p>2. 개인정보의 수집 및 이용목적</p>
          <p>- 정책 지원 신청 및 관리</p>
          <p>- 지원자와의 연락 및 안내</p>
          <p>3. 개인정보의 보유 및 이용기간</p>
          <p>- 신청일로부터 1년</p>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="indigo-darken-2"
            text
            @click="showPrivacyDialog = false"
          >
            닫기
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- 제출 완료 스낵바 -->
    <v-snackbar
      v-model="showSnackbar"
      :timeout="3000"
      color="success"
    >
      신청이 완료되었습니다.
      <template v-slot:action="{ attrs }">
        <v-btn
          text
          v-bind="attrs"
          @click="showSnackbar = false"
        >
          닫기
        </v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script setup lang="ts">
// 이전 스크립트 코드와 동일
import { ref, reactive } from 'vue';

const form = ref(null);
const isFormValid = ref(false);
const showPrivacyDialog = ref(false);
const showSnackbar = ref(false);

const genderOptions = ['남성', '여성', '기타'];

const machineOptions = [
  { text: '스마트워치', value: 'smart_watch' },
  { text: 'AI 인형', value: 'ai_doll' },
];

const formData = reactive({
  name: '',
  gender: '',
  age: '',
  reason: '',
  expectation: '',
  machines: [],
  phone: '',
  agreement: false,
});

const rules = {
  required: (v: any) => !!v || '필수 입력 항목입니다.',
  ageLimit: (v: any) => (v >= 0 && v <= 99) || '나이를 정확하게 입력해주세요',
  minLength: (v: any) => v.length >= 30 || '최소 30자 이상 입력해주세요.',
  phoneFormat: (v: any) => /^010-\d{4}-\d{4}$/.test(v) || '올바른 전화번호 형식이 아닍니다.',
  agreement: (v: any) => v || '개인정보 수집 및 이용에 동의해야 합니다.',
  atLeastOne: (v: any) => v.length > 0 || '하나 이상의 기계를 선택해주세요.',
};

const submitForm = async () => {
  const formElement = form.value as any;
  const isValid = await formElement?.validate();
  
  if (isValid) {
    console.log('Form submitted:', formData);
    showSnackbar.value = true;
    
    formElement?.reset();
    Object.keys(formData).forEach(key => {
      if (Array.isArray(formData[key])) {
        formData[key] = [];
      } else if (typeof formData[key] === 'boolean') {
        formData[key] = false;
      } else {
        formData[key] = '';
      }
    });
  }
};
</script>

<style scoped>
.cursor-pointer {
  cursor: pointer;
}
</style>