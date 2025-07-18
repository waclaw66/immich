<script lang="ts">
  import UserPageLayout from '$lib/components/layouts/user-page-layout.svelte';
  import AddToAlbum from '$lib/components/photos-page/actions/add-to-album.svelte';
  import ArchiveAction from '$lib/components/photos-page/actions/archive-action.svelte';
  import ChangeDate from '$lib/components/photos-page/actions/change-date-action.svelte';
  import ChangeDescription from '$lib/components/photos-page/actions/change-description-action.svelte';
  import ChangeLocation from '$lib/components/photos-page/actions/change-location-action.svelte';
  import CreateSharedLink from '$lib/components/photos-page/actions/create-shared-link.svelte';
  import DeleteAssets from '$lib/components/photos-page/actions/delete-assets.svelte';
  import DownloadAction from '$lib/components/photos-page/actions/download-action.svelte';
  import FavoriteAction from '$lib/components/photos-page/actions/favorite-action.svelte';
  import SelectAllAssets from '$lib/components/photos-page/actions/select-all-assets.svelte';
  import SetVisibilityAction from '$lib/components/photos-page/actions/set-visibility-action.svelte';
  import TagAction from '$lib/components/photos-page/actions/tag-action.svelte';
  import AssetGrid from '$lib/components/photos-page/asset-grid.svelte';
  import AssetSelectControlBar from '$lib/components/photos-page/asset-select-control-bar.svelte';
  import ButtonContextMenu from '$lib/components/shared-components/context-menu/button-context-menu.svelte';
  import EmptyPlaceholder from '$lib/components/shared-components/empty-placeholder.svelte';
  import { AssetAction } from '$lib/constants';
  import { TimelineManager } from '$lib/managers/timeline-manager/timeline-manager.svelte';
  import { AssetInteraction } from '$lib/stores/asset-interaction.svelte';
  import { preferences } from '$lib/stores/user.store';
  import { mdiDotsVertical, mdiPlus } from '@mdi/js';
  import { onDestroy } from 'svelte';
  import { t } from 'svelte-i18n';
  import type { PageData } from './$types';

  interface Props {
    data: PageData;
  }

  let { data }: Props = $props();

  const timelineManager = new TimelineManager();
  void timelineManager.updateOptions({ isFavorite: true, withStacked: true });
  onDestroy(() => timelineManager.destroy());

  const assetInteraction = new AssetInteraction();

  const handleEscape = () => {
    if (assetInteraction.selectionActive) {
      assetInteraction.clearMultiselect();
      return;
    }
  };

  const handleSetVisibility = (assetIds: string[]) => {
    timelineManager.removeAssets(assetIds);
    assetInteraction.clearMultiselect();
  };
</script>

<UserPageLayout hideNavbar={assetInteraction.selectionActive} title={data.meta.title} scrollbar={false}>
  <AssetGrid
    enableRouting={true}
    withStacked={true}
    {timelineManager}
    {assetInteraction}
    removeAction={AssetAction.UNFAVORITE}
    onEscape={handleEscape}
  >
    {#snippet empty()}
      <EmptyPlaceholder text={$t('no_favorites_message')} />
    {/snippet}
  </AssetGrid>
</UserPageLayout>

<!-- Multiselection mode app bar -->
{#if assetInteraction.selectionActive}
  <AssetSelectControlBar
    assets={assetInteraction.selectedAssets}
    clearSelect={() => assetInteraction.clearMultiselect()}
  >
    <FavoriteAction removeFavorite onFavorite={(assetIds) => timelineManager.removeAssets(assetIds)} />
    <CreateSharedLink />
    <SelectAllAssets {timelineManager} {assetInteraction} />
    <ButtonContextMenu icon={mdiPlus} title={$t('add_to')}>
      <AddToAlbum />
      <AddToAlbum shared />
    </ButtonContextMenu>
    <ButtonContextMenu icon={mdiDotsVertical} title={$t('menu')}>
      <DownloadAction menuItem />
      <ChangeDate menuItem />
      <ChangeDescription menuItem />
      <ChangeLocation menuItem />
      <ArchiveAction
        menuItem
        unarchive={assetInteraction.isAllArchived}
        onArchive={(assetIds) => timelineManager.removeAssets(assetIds)}
      />
      {#if $preferences.tags.enabled}
        <TagAction menuItem />
      {/if}
      <SetVisibilityAction menuItem onVisibilitySet={handleSetVisibility} />
      <DeleteAssets
        menuItem
        onAssetDelete={(assetIds) => timelineManager.removeAssets(assetIds)}
        onUndoDelete={(assets) => timelineManager.addAssets(assets)}
      />
    </ButtonContextMenu>
  </AssetSelectControlBar>
{/if}
