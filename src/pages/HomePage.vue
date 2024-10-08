<script setup lang="ts">
import { ref } from 'vue';
import QRCodeVue3 from 'qrcode-vue3';

import NavBar from '@/components/NavBar.vue';
import Main from '@/components/Main.vue';
import ShadowBox from '@/components/ShadowBox.vue';
import Badge from '@/components/Badge.vue';
import {
  Dialog,
  // DialogHeader,
  // DialogDescription,
  // DialogTitle,
  DialogContent,
  DialogFooter,
  DialogClose,
  DialogTrigger
} from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';

const showSummaryDetail = ref(false);

const handleSummaryDetail = () => {
  showSummaryDetail.value = !showSummaryDetail.value;
};

const isAuthenticated = ref(false);
const authenticate = async () => {
  if (isAuthenticated.value) {
    return;
  }
  if (!window.PublicKeyCredential) {
    // authResult.value = 'Web Authentication API가 지원되지 않습니다.';
    return;
  }
  try {
    const options: object = {
      publicKey: {
        challenge: new Uint8Array(32), // 서버에서 생성된 챌린지를 사용해야 합니다
        rp: {
          name: '어플리케이션이름',
          id: location.hostname
        },
        user: {
          id: new Uint8Array(16), // 사용자 ID를 적절히 설정해야 합니다
          name: '유저명',
          displayName: '유저명'
        },
        pubKeyCredParams: [{ alg: -7, type: 'public-key' }],
        authenticatorSelection: {
          authenticatorAttachment: 'platform',
          userVerification: 'required'
        },
        timeout: 60000
      }
    };
    const credential = await navigator.credentials.create(options);
    // authResult.value = '인증 성공!';
    isAuthenticated.value = true;
    console.log('Credential', credential);
  } catch (error: unknown) {
    // authResult.value = `인증 실패: ${error}`;
    isAuthenticated.value = false;
    console.error('Authentication error', error);
  }
};
</script>

<template>
  <NavBar />
  <Main :headbar="false" :navbar="true" :padded="true" :bg-gray="true">
    <div class="notice">
      <img src="/images/tada.svg" />
      <div>아직 조제받지 않은 처방전이 있어요</div>
    </div>

    <ShadowBox :padding-x="20" :padding-y="20" @click="$router.push('/login')"
      >로그인페이지</ShadowBox
    >
    <ShadowBox :padding-x="20" :padding-y="20" @click="$router.push('/pharmacist')"
      >약사페이지</ShadowBox
    >

    <ShadowBox :height="130">
      <div class="ticket-left">
        <div>
          <div class="hospital-name">김성헌 내과의원</div>
          <div class="hospital-address">서울시 광진구 능동로 195-16</div>
        </div>
        <div class="ticket-date">
          <i class="fa-regular fa-calendar"></i>
          <div class="date-and-time">24. 09. 10</div>
        </div>
      </div>
      <Dialog>
        <DialogTrigger>
          <div class="ticket-right" @click="authenticate">
            <img src="/images/qr-logo.png" alt="" />
            <div class="qr-text">약 받기</div>
          </div>
        </DialogTrigger>
        <DialogContent>
          <div class="qr-content-frame" v-if="isAuthenticated">
            <QRCodeVue3
              value="{userid=1,docno=12}"
              :qrOptions="{ typeNumber: 0, mode: 'Byte', errorCorrectionLevel: 'H' }"
              :imageOptions="{ hideBackgroundDots: false, imageSize: 0, margin: 0 }"
              :dotsOptions="{
                type: 'square',
                color: '#000000',
                gradient: {
                  type: 'linear',
                  rotation: 0,
                  colorStops: [
                    { offset: 0, color: '#000000' },
                    { offset: 1, color: '#000000' }
                  ]
                }
              }"
              :backgroundOptions="{ color: '#ffffff' }"
              :cornersSquareOptions="{ type: 'square', color: '#000000' }"
              :cornersDotOptions="{ type: undefined, color: '#000000' }"
              fileExt="png"
              myclass="my-qur"
              imgclass="img-qr"
            />
          </div>
          <div class="text-center mt-4" v-else>본인인증을 완료해주세요</div>
          <DialogFooter class="modal-footer">
            <DialogClose>
              <Button variant="destructive" size="lg">닫기</Button>
            </DialogClose>
          </DialogFooter>
        </DialogContent>
      </Dialog>
    </ShadowBox>

    <ShadowBox :padding-x="20" :padding-y="20">
      <div class="shadow-box-title">오늘 복용 확인</div>
      <div class="daily-check-container">
        <div class="daily-check">
          <div class="daily-check-left">
            <div class="daily-check-text">아침 식사 후 알약 3개</div>
            <Badge>졸음</Badge>
          </div>
          <Button variant="destructive">취소</Button>
        </div>
        <div class="daily-check">
          <div class="daily-check-left">
            <div class="daily-check-text">점심 식사 후 알약 3개</div>
            <Badge>졸음</Badge>
          </div>
          <Button variant="destructive">취소</Button>
        </div>
        <div class="daily-check">
          <div class="daily-check-left">
            <div class="daily-check-text">저녁 식사 후 알약 3개</div>
            <Badge>졸음</Badge>
          </div>
          <Button>확인</Button>
        </div>
      </div>
    </ShadowBox>

    <ShadowBox :padding-x="20" :padding-y="20">
      <div>
        <div class="title-with-arrow" @click="handleSummaryDetail()">
          <div class="shadow-box-title">최규찬님,</div>
          <i class="fa-solid fa-chevron-down" :class="{ rotate: showSummaryDetail }"></i>
        </div>
        <div v-if="showSummaryDetail">
          <div class="tab-select-container">
            <span class="tab" :class="{ selected: true }">최규찬 정형외과의원</span>
            <span class="tab">최규찬 정형외과의원</span>
            <span class="tab">최규찬 정형외과의원</span>
            <span class="tab">최규찬 정형외과의원</span>
            <span class="tab">최규찬 정형외과의원</span>
          </div>
          <div>
            <div class="report-top">
              <div class="flex">
                <div class="report-icon">
                  <img src="/images/report-pill.svg" alt="" />
                </div>
                <div class="flex flex-col justify-center">
                  <span class="report-title">약 복용방법</span>
                  <span class="date-and-time">어떤 설명인지에 대한 설명</span>
                </div>
              </div>
              <img src="/images/report-speaker.svg" alt="" />
            </div>
            <div class="report-content">
              1일 3회, 식후 30분에 물과 함께 복용하세요. 3일간 꾸준히 드시고, 증상이 지속되면 의사와
              상담하세요. 약을 삼키기 어려우면 물에 녹여 드셔도 됩니다.
            </div>
          </div>
        </div>
        <div v-else class="summary-short-text">이것저것하고 이것저것 활동하고 이것저것 드세요</div>
      </div>
    </ShadowBox>

    <ShadowBox :padding-x="20" :padding-y="20">
      <div class="title-with-arrow">
        <div class="shadow-box-title">최근 처방전 내역</div>
        <i class="fa-solid fa-chevron-right"></i>
      </div>
      <div class="prescription-container">
        <div>
          <div class="flex justify-between">
            <div class="flex items-center">
              <div class="middot"></div>
              <span class="history-title">최규찬 정형외과의원</span>
            </div>
            <div class="history-title" :class="{ received: false }">수령완료</div>
          </div>
          <div class="history-date">24. 09. 10 | 오후 4:18</div>
        </div>
        <div>
          <div class="flex justify-between">
            <div class="flex items-center">
              <div class="middot"></div>
              <span class="history-title">최규찬 정형외과의원</span>
            </div>
            <div class="history-title" :class="{ received: true }">수령완료</div>
          </div>
          <div class="history-date">24. 09. 10 | 오후 4:18</div>
        </div>
      </div>
    </ShadowBox>
  </Main>
</template>

<style scoped>
.notice {
  margin-top: 20px;
  display: flex;
  align-items: center;
  font-size: 12px;
  color: var(--dark-gray);
  margin-bottom: 4px;
  gap: 2px;
}

.ticket-left {
  position: relative;
  width: 70%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 16px 20px;
}

.ticket-right {
  position: absolute;
  top: 0;
  right: 0;
  height: 100%;
  width: 30%;
  background-color: var(--css-primary);
  border-radius: 0 10px 10px 0;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.hospital-name {
  font-size: 20px;
  font-weight: 600;
  margin-bottom: 2px;
}

.hospital-address {
  font-size: 14px;
  color: var(--dark-gray);
}

.date-and-time {
  font-size: 12px;
  color: var(--dark-gray);
}

.ticket-date {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  columns: var(--dark-gray);
}

.fa-calendar {
  color: var(--dark-gray);
}

.qr-text {
  font-size: 16px;
  font-weight: 600;
  margin-top: 8px;
}

.shadow-box-title {
  font-weight: 600;
  font-size: 18px;
}

.daily-check-container {
  margin-top: 16px;
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 28px;
}

.daily-check {
  display: flex;
  justify-content: space-between;
}

.daily-check-left {
  display: flex;
  align-items: center;
  gap: 8px;
}

.daily-check-text {
  font-size: 16px;
}

.title-with-arrow {
  display: flex;
  width: 100%;
  justify-content: space-between;
  align-items: center;
}

.summary-short-text {
  margin-top: 16px;
  font-size: 16px;
}

.prescription-container {
  display: flex;
  flex-direction: column;
  margin-top: 16px;
  gap: 16px;
}

.fa-chevron-down {
  transition: transform 0.3s ease;
}

.fa-chevron-down.rotate {
  transform: rotate(180deg);
}

.tab-select-container {
  display: flex;
  margin: 16px 0;
  width: calc(100% + 20px);
  gap: 8px;
  overflow-x: scroll;
  white-space: nowrap;
}

.tab-select-container::-webkit-scrollbar {
  display: none !important;
}

.tab {
  font-size: 12px;
  border-radius: 20px;
  color: var(--css-muted);
  border: 1px solid var(--css-muted);
  padding: 4px 12px;
  transition: all 0.2s ease;
}

.tab:hover {
  background-color: var(--css-muted);
  color: var(--white);
}

.tab.selected {
  background-color: var(--css-muted);
  color: var(--white);
}

.report-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.report-icon {
  width: 44px;
  height: 44px;
  border-radius: 6px;
  background-color: #febbb1;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-right: 8px;
}

.report-title {
  font-size: 16px;
  font-weight: 600;
}

.report-content {
  font-size: 15px;
}

.middot {
  width: 6px;
  height: 6px;
  border-radius: 100px;
  background-color: var(--black);
  margin-right: 6px;
}

.history-title {
  font-size: 16px;
  font-weight: 500;
}

.history-title.received {
  color: var(--red);
}

.history-date {
  margin-left: 12px;
  font-size: 12px;
  color: var(--dark-gray);
}

.qr-content-frame {
  margin-top: 20px;
  margin-bottom: 4px;
  width: 100%;
  display: flex;
  justify-content: center;
  padding: 8px;
  border: 1px solid var(--gray);
  border-radius: 4px;
}

.modal-footer {
  margin-top: 8px;
  display: flex;
  flex-direction: column;
}
</style>
