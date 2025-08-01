<template>
    <div class="full-height-scrollable">
        <loading
            :loading="isLoading"
            lazy
        >
            <div class="card-body">
                <div class="d-flex">
                    <div class="flex-fill">
                        <nav aria-label="breadcrumb">
                            <ol class="breadcrumb">
                                <li class="breadcrumb-item">
                                    <router-link :to="{name: 'public:podcasts'}">
                                        {{ $gettext('Podcasts') }}
                                    </router-link>
                                </li>
                                <li class="breadcrumb-item">
                                    {{ podcast.title }}
                                </li>
                            </ol>
                        </nav>

                        <podcast-common :podcast="podcast" />

                        <div class="buttons">
                            <a
                                class="btn btn-warning btn-sm"
                                :href="podcast.links.public_feed"
                                target="_blank"
                            >
                                <icon :icon="IconRss" />
                                {{ $gettext('RSS') }}
                            </a>
                        </div>
                    </div>
                    <div class="flex-shrink ps-3">
                        <album-art
                            :src="podcast.art"
                            :width="128"
                        />
                    </div>
                </div>
            </div>
        </loading>

        <data-table
            v-if="groupLayout === 'table'"
            id="podcast-episodes"
            paginated
            :fields="fields"
            :provider="episodesItemProvider"
        >
            <template #cell(play_button)="{item}">
                <play-button
                    icon-class="lg"
                    :url="item.links.download"
                />
            </template>
            <template #cell(art)="{item}">
                <album-art
                    :src="item.art"
                    :width="64"
                />
            </template>
            <template #cell(title)="{item}">
                <h5 class="m-0">
                    <router-link
                        :to="{name: 'public:podcast:episode', params: {podcast_id: podcast.id, episode_id: item.id}}"
                    >
                        {{ item.title }}
                    </router-link>
                </h5>
                <div class="badges my-2">
                    <span
                        v-if="item.publish_at"
                        class="badge text-bg-secondary"
                    >
                        {{ formatTimestampAsDateTime(item.publish_at) }}
                    </span>
                    <span
                        v-else
                        class="badge text-bg-secondary"
                    >
                        {{ formatTimestampAsDateTime(item.created_at) }}
                    </span>
                    <span
                        v-if="item.explicit"
                        class="badge text-bg-danger"
                    >
                        {{ $gettext('Explicit') }}
                    </span>
                </div>
                <p class="card-text">
                    {{ item.description_short }}
                </p>
            </template>
            <template #cell(actions)="{item}">
                <div class="btn-group btn-group-sm">
                    <router-link
                        :to="{name: 'public:podcast:episode', params: {podcast_id: podcast.id, episode_id: item.id}}"
                        class="btn btn-primary"
                    >
                        {{ $gettext('Details') }}
                    </router-link>
                </div>
            </template>
        </data-table>
        <grid-layout
            v-else
            id="podcast-episodes-grid"
            ref="$grid"
            paginated
            :api-url="episodesUrl"
        >
            <template #item="{item}: {item: ApiPodcastEpisode}">
                <div class="card mb-4">
                    <div class="card-header d-flex align-items-center">
                        <div class="flex-shrink-0 pe-2">
                            <play-button
                                icon-class="lg"
                                :url="item.links.download"
                            />
                        </div>
                        <h5 class="card-title flex-fill m-0">
                            <router-link
                                :to="{name: 'public:podcast:episode', params: {podcast_id: podcast.id, episode_id: item.id}}"
                            >
                                {{ item.title }}
                            </router-link>
                        </h5>
                        <div class="flex-shrink-0 ps-2">
                            <album-art
                                :src="item.art"
                                :width="64"
                            />
                        </div>
                    </div>
                    <div class="card-body">
                        <div class="badges my-2">
                            <span
                                v-if="item.publish_at"
                                class="badge text-bg-secondary"
                            >
                                {{ formatTimestampAsDateTime(item.publish_at) }}
                            </span>
                            <span
                                v-else
                                class="badge text-bg-secondary"
                            >
                                {{ formatTimestampAsDateTime(item.created_at) }}
                            </span>
                            <span
                                v-if="item.explicit"
                                class="badge text-bg-danger"
                            >
                                {{ $gettext('Explicit') }}
                            </span>
                        </div>
                        <p class="card-text">
                            {{ item.description_short }}
                        </p>
                    </div>
                </div>
            </template>
        </grid-layout>
    </div>
</template>

<script setup lang="ts">
import {getStationApiUrl} from "~/router.ts";
import {useRoute} from "vue-router";
import DataTable, {DataTableField} from "~/components/Common/DataTable.vue";
import Loading from "~/components/Common/Loading.vue";
import AlbumArt from "~/components/Common/AlbumArt.vue";
import {useTranslate} from "~/vendor/gettext.ts";
import {IconRss} from "~/components/Common/icons.ts";
import Icon from "~/components/Common/Icon.vue";
import PlayButton from "~/components/Common/PlayButton.vue";
import useStationDateTimeFormatter from "~/functions/useStationDateTimeFormatter.ts";
import PodcastCommon from "~/components/Public/Podcasts/PodcastCommon.vue";
import GridLayout from "~/components/Common/GridLayout.vue";
import {ApiPodcastEpisode} from "~/entities/ApiInterfaces.ts";
import {usePodcastGlobals} from "~/components/Public/Podcasts/usePodcastGlobals.ts";
import {computed} from "vue";
import {useApiItemProvider} from "~/functions/dataTable/useApiItemProvider.ts";
import {QueryKeys} from "~/entities/Queries.ts";
import {usePodcastQuery} from "~/components/Public/Podcasts/usePodcastQuery.ts";

const {groupLayout, stationId, stationTz} = usePodcastGlobals();

const {params} = useRoute();

const {data: podcast, isLoading} = usePodcastQuery();

const {$gettext} = useTranslate();
const fields: DataTableField<ApiPodcastEpisode>[] = [
    {key: 'play_button', label: '', sortable: false, class: 'shrink pe-0'},
    {key: 'art', label: '', sortable: false, class: 'shrink pe-0'},
    {key: 'title', label: $gettext('Episode'), sortable: true},
    {key: 'actions', label: $gettext('Actions'), sortable: false, class: 'shrink'}
];

const episodesUrl = getStationApiUrl(computed(() => {
    const podcastId = params.podcast_id as string;
    return `/public/podcast/${podcastId}/episodes`;
}), stationId);

const episodesItemProvider = useApiItemProvider<ApiPodcastEpisode>(
    episodesUrl,
    [
        QueryKeys.PublicPodcasts,
        {station: stationId},
        params.podcast_id,
        'episodes'
    ],
    {
        staleTime: 5 * 60 * 1000
    }
);

const {formatTimestampAsDateTime} = useStationDateTimeFormatter(stationTz);
</script>
