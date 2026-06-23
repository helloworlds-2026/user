<template>
  <div class="min-h-screen bg-background text-foreground pt-24 pb-16">
    <div class="container mx-auto px-4">
      <div class="mb-8 flex items-center justify-between">
        <div>
          <h1 class="text-3xl font-black text-foreground mb-2">{{ t('rechargeOrder.title') }}</h1>
          <p class="text-muted-foreground text-sm">{{ t('rechargeOrder.subtitle') }}</p>
        </div>
        <router-link to="/me/orders" class="text-muted-foreground transition-colors hover:text-foreground text-sm">{{ t('rechargeOrder.backList') }}</router-link>
      </div>

      <div v-if="loading" class="h-40 border bg-muted rounded-2xl animate-pulse"></div>

      <EmptyState
        v-else-if="!recharge"
        icon="alert"
        :title="t('rechargeOrder.notFound')"
        :action-label="t('errorBoundary.retry')"
        @action="loadDetail()"
      />

      <div v-else class="space-y-6">
        <!-- 头部信息 -->
        <div class="rounded-2xl border bg-card shadow-sm p-6">
          <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-4">
            <div>
              <div class="text-xs uppercase tracking-wider text-muted-foreground">{{ t('personalCenter.wallet.rechargeNoLabel') }}</div>
              <div class="text-sm font-semibold text-foreground mt-1">{{ recharge.recharge_no }}</div>
              <div class="text-xs text-muted-foreground mt-2 flex flex-wrap items-center gap-x-4 gap-y-1">
                <span>{{ t('rechargeOrder.createdAtLabel') }}：{{ formatDate(recharge.created_at) }}</span>
                <span v-if="paymentExpiresAt">{{ t('payment.expiresAt') }}：{{ formatDate(paymentExpiresAt) }}</span>
              </div>
            </div>
            <div class="flex flex-col items-start md:items-end gap-2">
              <div class="text-xs uppercase tracking-wider text-muted-foreground">{{ t('rechargeOrder.rechargeAmount') }}</div>
              <div class="text-lg font-bold text-foreground">{{ formatMoney(recharge.amount, recharge.currency) }}</div>
            </div>
            <div class="flex items-center gap-3">
              <Badge :variant="rechargeStatusVariant(recharge.status)" size="sm">
                {{ rechargeStatusText(recharge.status) }}
              </Badge>
            </div>
          </div>
        </div>

        <!-- 金额明细 -->
        <div class="rounded-2xl border bg-card shadow-sm p-6">
          <h2 class="text-lg font-bold mb-4">{{ t('rechargeOrder.amountTitle') }}</h2>
          <div class="grid grid-cols-1 md:grid-cols-4 gap-4 text-sm">
            <div class="border rounded-xl p-4">
              <div class="text-xs text-muted-foreground">{{ t('rechargeOrder.rechargeAmount') }}</div>
              <div class="text-foreground font-mono mt-1">{{ formatMoney(recharge.amount, recharge.currency) }}</div>
            </div>
            <div class="border rounded-xl p-4">
              <div class="text-xs text-muted-foreground">{{ t('payment.feeRateLabel') }}</div>
              <div class="text-foreground font-mono mt-1">{{ feeRateDisplay }}</div>
            </div>
            <div class="border rounded-xl p-4">
              <div class="text-xs text-muted-foreground">{{ t('payment.feeAmountLabel') }}</div>
              <div class="text-foreground font-mono mt-1">{{ formatMoney(recharge.fee_amount, recharge.currency) }}</div>
            </div>
            <div class="border rounded-xl p-4">
              <div class="text-xs text-muted-foreground">{{ t('personalCenter.wallet.payAmountLabel') }}</div>
              <div class="text-foreground font-mono mt-1 font-bold">{{ formatMoney(recharge.payable_amount, recharge.currency) }}</div>
            </div>
          </div>
        </div>

        <!-- 支付区域（仅待支付状态） -->
        <div v-if="isPending" class="rounded-2xl border bg-card shadow-sm p-6">
          <h2 class="text-lg font-bold mb-4">{{ t('rechargeOrder.paymentTitle') }}</h2>
          <div v-if="isPending" class="mb-3 text-xs text-muted-foreground">
            {{ t('personalCenter.wallet.pendingHint') }}
          </div>
          <div class="flex flex-col items-center gap-4">
            <div v-if="showQRCode" class="w-full max-w-sm rounded-xl border p-4">
              <div class="mb-3 text-sm font-semibold text-foreground text-center">{{ t('payment.qrTitle') }}</div>
              <div class="flex items-center justify-center">
                <img :src="qrImageUrl" alt="Recharge QR" class="h-52 w-52 object-contain" />
              </div>
              <div v-if="qrUsingPayLinkFallback" class="mt-3 text-xs text-muted-foreground text-center">
                {{ t('payment.qrFallbackHint') }}
              </div>
              <div class="mt-4 flex justify-center">
                <Button type="button" variant="outline" size="sm" :disabled="checkingPayment" @click="checkPayment">
                  {{ checkingPayment ? t('personalCenter.wallet.checkingPayStatus') : t('personalCenter.wallet.checkPayStatus') }}
                </Button>
              </div>
            </div>

            <div v-if="hasCryptoPaymentDetails" class="w-full max-w-md rounded-xl border p-4">
              <div class="space-y-2 rounded-xl border bg-muted/40 p-3 text-sm">
                <div
                  v-for="item in cryptoPaymentDetails"
                  :key="item.key"
                  class="flex flex-col gap-1 border-b pb-2 last:border-b-0 last:pb-0"
                >
                  <span class="text-xs text-muted-foreground">{{ item.label }}</span>
                  <span class="min-w-0 font-semibold text-foreground break-all">
                    {{ item.value }}
                    <span v-if="item.detail" class="ml-1 font-normal text-muted-foreground">({{ item.detail }})</span>
                  </span>
                </div>
                <div v-if="cryptoWalletAddress" class="flex flex-wrap items-center gap-2 pt-1">
                  <Button type="button" variant="outline" size="sm" @click="handleCopyWalletAddress">
                    {{ t('payment.copyWalletAddress') }}
                  </Button>
                  <span v-if="walletAddressCopied" class="text-xs text-emerald-500">{{ t('payment.copied') }}</span>
                </div>
              </div>
            </div>

            <div v-if="!showQRCode" class="flex justify-center">
              <Button type="button" variant="outline" size="sm" :disabled="checkingPayment" @click="checkPayment">
                {{ checkingPayment ? t('personalCenter.wallet.checkingPayStatus') : t('personalCenter.wallet.checkPayStatus') }}
              </Button>
            </div>

            <div v-if="showTelegramPayHint" class="text-xs text-muted-foreground text-center">
              {{ t('payment.telegramExternalHint') }}
            </div>
          </div>
        </div>

        <!-- 支付成功提示 -->
        <div v-if="recharge.status === 'success'" class="rounded-2xl border bg-card shadow-sm p-6 border-l-4 border-green-500">
          <p class="text-sm font-semibold text-foreground">{{ t('personalCenter.wallet.rechargeSuccess') }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import EmptyState from '../components/EmptyState.vue'
import { useRechargeOrderDetail } from '../composables/useRechargeOrderDetail'

const { t } = useI18n()

const {
  loading, checkingPayment, recharge, walletAddressCopied, qrImageUrl,
  isPending, paymentExpiresAt, showTelegramPayHint, qrUsingPayLinkFallback, showQRCode,
  cryptoWalletAddress, cryptoPaymentDetails, hasCryptoPaymentDetails, feeRateDisplay,
  rechargeStatusText, rechargeStatusVariant, formatMoney, formatDate,
  loadDetail, checkPayment, handleCopyWalletAddress,
} = useRechargeOrderDetail()
</script>
