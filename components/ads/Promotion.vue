<template>
  <div v-if="!$cosmetics.hideAds" class="MYYLVTXBPUVWMLVBPVSDLHADDRYFBF">
    <div class="MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-0">
      <div class="MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-1">
        <div class="MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-2">
          <a
            href="https://exaroton.com/?utm_source=modrinth&utm_medium=text&utm_campaign=host&utm_content=top"
            rel="noopener nofollow sponsored"
            target="_blank"
          >
            <LightIcon
              v-if="colorMode.value === 'light'"
              class="MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-3"
            />
            <DarkIcon v-else class="MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-3" />
            <span>
              <span> Host your Minecraft server on </span>
              <strong>exaroton</strong>
              <span> - only pay while the server is running - billed per second. </span>
            </span>
          </a>
        </div>
      </div>
      <div class="MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-4">
        <a rel="noopener sponsored" target="_blank" href="https://adrinth.com"> Ads via Adrinth </a>
      </div>
    </div>
  </div>
</template>
<script setup>
import LightIcon from '~/assets/images/external/exaroton-light.svg'
import DarkIcon from '~/assets/images/external/exaroton-dark.svg'

const colorMode = useTheme()
</script>

<style>
.MYYLVTXBPUVWMLVBPVSDLHADDRYFBF {
  position: relative;
  margin-bottom: var(--spacing-card-md);
  background: var(--color-ad);
  border: 3px solid var(--color-ad-raised);
  border-radius: var(--size-rounded-card);
}
.MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-0 {
  font-size: 14px;
  line-height: 1.3em;
}
.MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-1 {
  color: var(--color-text);
  padding: 1em;
  text-align: left;
}
.MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-2 a {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-2 a b,
.MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-2 a strong {
  color: var(--color-brand-secondary);
}
.MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-3 {
  padding-top: 1px;
  height: 1.2rem;
  width: auto;
}
.MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-4 a {
  position: absolute;
  bottom: -2px;
  right: -2px;
  text-align: center;
  font-weight: 600;
  text-transform: uppercase;
  font-size: 0.8em;
  background: var(--color-ad-raised);
  letter-spacing: 0.1ch;
  margin: 0;
  padding: 2px 10px;
  border-top-left-radius: var(--size-rounded-card);
  border-bottom-right-radius: var(--size-rounded-card);
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: black;
}
@media screen and (max-width: 800px) {
  .MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-2 {
    margin-bottom: 0.5rem;
  }
  .MYYLVTXBPUVWMLVBPVSDLHADDRYFBF-2 a {
    align-items: flex-start;
    flex-direction: column;
  }
}
</style>
