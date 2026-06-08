<script lang="ts">
  import type {
    HTMLButtonAttributes,
    HTMLAnchorAttributes,
  } from 'svelte/elements'
  import { tooltip } from 'tint/actions'

  type Props = HTMLButtonAttributes &
    HTMLAnchorAttributes & {
      icon?: boolean
      toggled?: boolean | undefined
      tooltip?: string | undefined
      external?: boolean
      disabled?: boolean
      submit?: boolean
      title?: string | undefined
      'aria-label'?: string | undefined
      tabindex?: number | undefined
      role?: string | undefined
      'aria-checked'?: boolean | undefined
      element?:
        | HTMLButtonElement
        | HTMLAnchorElement
        | HTMLSpanElement
        | undefined
      children?: import('svelte').Snippet
      onclick?: (e: MouseEvent) => void
      class?: string
    }

  let {
    icon = false,
    toggled = undefined,
    href = undefined,
    external = false,
    download = undefined,
    disabled = false,
    submit = false,
    formmethod = undefined,
    title = undefined,
    'aria-label': ariaLabel = undefined,
    tabindex = undefined,
    role: roleOverride = undefined,
    'aria-checked': ariaChecked = undefined,
    element = $bindable(undefined),
    children,
    onclick = undefined,
    onkeypress = undefined,
    onkeydown = undefined,
    tooltip: tooltipText = undefined,
    class: className = '',
    ...elementProps
  }: Props = $props()

  $effect.pre(() => {
    if (icon && !title && !ariaLabel && !tooltipText) {
      throw new Error(
        '[BumpButton] Icon buttons need at least a title, aria-label, or tooltip',
      )
    }
    if (href && toggled !== undefined) {
      throw new Error('[BumpButton] Links cannot be toggled')
    }
  })

  let role = $derived(
    roleOverride || (toggled !== undefined ? 'switch' : undefined),
  )
  let ariaPressed = $derived(toggled !== undefined ? toggled : undefined)
</script>

{#if href && disabled}
  <span
    {title}
    aria-disabled="true"
    aria-label={ariaLabel}
    bind:this={element}
    use:tooltip={tooltipText}
    class:icon
    class:toggled
    class={`bump-button tint--type-body-sans-bold ${className}`}
    {...elementProps as Record<string, unknown>}>{@render children?.()}</span
  >
{:else if href}
  <a
    {download}
    {href}
    {tabindex}
    {title}
    aria-label={ariaLabel}
    bind:this={element}
    use:tooltip={tooltipText}
    class:icon
    class={`bump-button tint--type-body-sans-bold ${className}`}
    rel={external ? 'noopener' : undefined}
    target={external ? '_blank' : undefined}
    {...elementProps}>{@render children?.()}</a
  >
{:else}
  <button
    {disabled}
    {role}
    {tabindex}
    {title}
    {formmethod}
    aria-label={ariaLabel}
    aria-pressed={ariaPressed}
    aria-checked={ariaChecked}
    bind:this={element}
    class:icon
    class:toggled
    class={`bump-button tint--type-body-sans-bold ${className}`}
    {onclick}
    {onkeypress}
    {onkeydown}
    use:tooltip={tooltipText}
    type={submit ? 'submit' : 'button'}
    {...elementProps}
  >
    {@render children?.()}
  </button>
{/if}

<style lang="sass">
  .bump-button
    display: inline-flex
    align-items: center
    justify-content: center
    box-sizing: border-box
    vertical-align: top
    background: var(--tint-bg)
    color: var(--tint-text)
    border: 4px solid currentColor
    border-radius: 999px
    padding: tint.$size-12 tint.$size-24
    text-decoration: none
    cursor: pointer
    box-shadow: 0 6px 0 0 currentColor
    transition: transform 80ms ease-out, box-shadow 80ms ease-out
    margin-block-end: 6px
    flex-shrink: 0
    min-height: 56px
    @-moz-document url-prefix()
      // This attempts to cover up a rendering bug in Firefox
      outline: 2px solid currentColor
      outline-offset: -1px

    &:focus-visible
      outline: 2px solid var(--tint-action-primary)
      outline-offset: -8px

    &.icon
      padding: 0
      width: 56px
      height: 56px

    &.toggled
      background: var(--tint-text)
      color: var(--tint-bg)

    & > :global(*)
      pointer-events: none

  a.bump-button, button.bump-button
    &:not(:disabled):not([aria-disabled="true"]):hover, &:not(:disabled):not([aria-disabled="true"]):focus-visible
      transform: translateY(2px)
      box-shadow: 0 4px 0 0 currentColor
    &:not(:disabled):not([aria-disabled="true"]):active
      transform: translateY(6px)
      box-shadow: 0 0 0 0 currentColor

  button.bump-button:disabled, span.bump-button[aria-disabled="true"]
    cursor: initial
    color: var(--tint-text-disabled)
    box-shadow: 0 6px 0 0 currentColor

  @media (forced-colors: active)
    .bump-button
      forced-color-adjust: none
      background-color: ButtonFace
      border-color: ButtonText
      color: ButtonText
      box-shadow: 0 6px 0 0 ButtonText
    button.bump-button:disabled, span.bump-button[aria-disabled="true"]
      opacity: 1
      background-color: ButtonFace
      border-color: GrayText
      color: GrayText
      box-shadow: 0 6px 0 0 GrayText
    .bump-button:focus-visible
      outline-color: CanvasText
</style>
