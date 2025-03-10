<script lang="ts">
  import { onDestroy,onMount } from 'svelte';
  import type { Address } from 'viem';

  import { Icon } from '$components/Icon';
  import Erc20 from '$components/Icon/ERC20.svelte';
  import { OnAccount } from '$components/OnAccount';
  import { tokenService } from '$libs/storage/services';
  import type { Token } from '$libs/token';
  import { classNames } from '$libs/util/classNames';
  import { noop } from '$libs/util/noop';
  import { account } from '$stores/account';

  import AddCustomErc20 from './AddCustomERC20.svelte';
  import { symbolToIconMap } from './symbolToIconMap';

  export let id: string;
  export let menuOpen = false;
  export let tokens: Token[] = [];
  export let customTokens: Token[] = [];
  export let value: Maybe<Token> = null;
  export let selectToken: (token: Token) => void = noop;

  let modalOpen = false;

  $: menuClasses = classNames(
    'menu absolute right-0 w-[265px] p-3 mt-2 rounded-[10px] bg-neutral-background z-10',
    menuOpen ? 'visible opacity-100' : 'invisible opacity-0',
  );

  const getTokenKeydownHandler = (token: Token) => {
    return (event: KeyboardEvent) => {
      if (event.key === 'Enter') {
        selectToken(token);
      }
    };
  };

  const showAddERC20 = () => (modalOpen = true);

  const handleStorageChange = (newTokens: Token[]) => {
    customTokens = newTokens;
  };

  const onAccountChange = () => {
    if ($account?.address) {
      customTokens = tokenService.getTokens($account?.address as Address);
    }
  };

  onMount(() => {
    tokenService.subscribeToChanges(handleStorageChange);
  });

  onDestroy(() => {
    tokenService.unsubscribeFromChanges(handleStorageChange);
  });
</script>

<!-- Desktop (or larger) view -->
<ul role="listbox" {id} class={menuClasses}>
  {#each tokens as token (token.symbol)}
    <li
      role="option"
      tabindex="0"
      aria-selected={token === value}
      on:click={() => selectToken(token)}
      on:keydown={getTokenKeydownHandler(token)}>
      <div class="p-4">
        <i role="img" aria-label={token.name}>
          <svelte:component this={symbolToIconMap[token.symbol]} />
        </i>
        <span class="body-bold">{token.symbol}</span>
      </div>
    </li>
  {/each}
  {#each customTokens as token, index (index)}
    <li
      role="option"
      tabindex="0"
      aria-selected={token === value}
      on:click={() => selectToken(token)}
      on:keydown={getTokenKeydownHandler(token)}>
      <div class="p-4">
        <i role="img" aria-label={token.name}>
          <Erc20 />
        </i>
        <span class="body-bold">{token.symbol}</span>
      </div>
    </li>
  {/each}
  <div class="h-sep" />
  <li>
    <button on:click={showAddERC20} class="flex hover:bg-dark-5 flex justify-center items-center p-4 rounded-sm">
      <Icon type="plus-circle" fillClass="fill-primary-icon" size={20} vWidth={30} vHeight={30} />
      <span
        class="
            text-sm
            font-medium
            bg-transparent
            flex-1
            w-[100px]
            px-0
            pl-2">
        Add Custom
      </span>
    </button>
  </li>
</ul>

<AddCustomErc20 bind:modalOpen />

<OnAccount change={onAccountChange} />
