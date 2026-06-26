<script lang="ts">
  import { browser } from "$app/environment";
  import { HugeiconsIcon } from "@hugeicons/svelte";
  import { Copy01Icon, CopyCheckIcon } from "@hugeicons/core-free-icons";
  import { onMount } from "svelte";

  interface StoredFeedback {
    reaction?: string;
    steering?: string;
    vote?: string;
  }

  interface Props {
    paragraphs: string[];
    run: string;
    slug: string;
    url: string;
    version: string;
    storagePrefix?: string;
    type?: string;
  }

  const {
    paragraphs,
    run,
    slug,
    url,
    version,
    storagePrefix = "tufte-mdsvx-feedback:",
    type = "tufte_mdsvx_feedback",
  }: Props = $props();
  let copied = $state(false);
  let copiedCount = $state(0);
  let copyButton: HTMLButtonElement;

  function readFeedback(paragraph: string) {
    if (!browser) return null;
    const raw = localStorage.getItem(`${storagePrefix}${paragraph}`);
    if (!raw) return null;
    try {
      const parsed = JSON.parse(raw) as StoredFeedback;
      const reaction = (parsed.reaction ?? parsed.vote ?? "").trim();
      const steering = (parsed.steering ?? "").trim();
      if (!reaction && !steering) return null;
      return {
        p: paragraph,
        ...(reaction ? { r: reaction } : {}),
        ...(steering ? { s: steering } : {}),
      };
    } catch {
      return null;
    }
  }

  function buildPayload() {
    const items = paragraphs.map(readFeedback).filter(Boolean);
    return {
      type,
      ref: { slug, version, run, url },
      items,
    };
  }

  async function copyText(text: string) {
    const textarea = document.createElement("textarea");
    textarea.value = text;
    textarea.setAttribute("readonly", "");
    textarea.style.position = "fixed";
    textarea.style.left = "-9999px";
    document.body.appendChild(textarea);
    textarea.select();
    const copiedWithSelection = document.execCommand("copy");
    textarea.remove();
    if (copiedWithSelection) return;

    if (navigator.clipboard?.writeText) {
      await navigator.clipboard.writeText(text);
    }
  }

  function copyFeedback() {
    const payload = buildPayload();
    copiedCount = payload.items.length;
    copied = true;
    void copyText(JSON.stringify(payload)).catch(() => {});
    window.setTimeout(() => (copied = false), 6000);
  }

  onMount(() => {
    copyButton.addEventListener("click", copyFeedback);
    return () => copyButton.removeEventListener("click", copyFeedback);
  });
</script>

<div class="feedback-export">
  <div>
    <p class="feedback-export-title">Feedback export</p>
    <p class="feedback-export-copy">Copies compact JSON keyed by page version and paragraph id. Paste it back to an agent and it can apply the notes without guessing.</p>
  </div>
  <button type="button" bind:this={copyButton}>
    <HugeiconsIcon icon={copied ? CopyCheckIcon : Copy01Icon} size={16} strokeWidth={1.8} />
    <span>{copied ? `copied ${copiedCount}` : "copy feedback"}</span>
  </button>
</div>
