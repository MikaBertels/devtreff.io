<static-query>
    query{
    allMainSection{
    edges {
    node {
    content {
    body
    title
    image
    _uid
    component
    image_alt_text
    }
    }
    }
    }
    allEdition{
    edges{
    node{
    path
    name
    slug
    full_slug
    nextEvent {
    name
    content {
    date
    hero_image
    }
    }
    content {
    name
    hero_image
    }
    }
    }
    }
    allEvent{
    edges{
    node{
    path
    name
    tag_list
    edition{
    name
    slug
    full_slug
    }
    content{
    date
    teaser_image
    blog_post {
    full_slug
    }
    location {
    name
    content {
    city
    }
    }
    }
    }
    }
    }
    }
</static-query>

<template>
    <FullImageLayout
            :has-gradient="true"
            alt="Eine Person stellt bei einem Vortrag eine Frage."
            image="//img2.storyblok.com/1024x600/f/59562/2048x1369/d54c0aa61b/76765482_474220090114834_8385444739406626816_o_474220086781501.jpg"
    >
        <template #hero>
            <div class="flex-1 flex justify-center items-center">
                <div class="max-w-4xl w-full mt-5 lg:mt-0">
                    <div class="max-w-2xl">
                        <h1 class="text-3xl md:text-5xl lg:text-6xl font-bold pb-3 lg:pb-8">DevTreff.io</h1>
                        <div class="card o2 p-5">
                            <p class="leading-relaxed lg:leading-loose">
                                Unter devTreff.io finden derzeit an 3 Standorten regelmäßige Treffen von interessierten
                                Entwicklern statt.
                                DevTreff.io ist ein "OpenSource" Projekt. Nicht nur diese Webseite, sondern auch die
                                Events leben von einer
                                aktiven Community. Es werden jederzeit, an allen Standorten SPEAKER gesucht, die ihr
                                Wissen mit einer
                                gleichgesinnten Gruppe teilen möchten.
                            </p>
                        </div>
                        <div class="mt-6 lg:mt-8 flex justify-between lg:flex-row flex-col">
                            <Button
                                    v-for="event in upcomingEvents"
                                    :key="event.uuid"
                                    tag="a"
                                    class="text-left flex-auto max-w-xs"
                                    :href="event.edition.slug"
                            >
                                <div class="font-bold">{{ event.content.location.content.city }}</div>
                                <div class="text-xs">
                                    <FormatDate :date-string="event.content.date"/>
                                </div>
                            </Button>
                        </div>

                    </div>
                </div>
            </div>
        </template>

        <template #main>

            <Section
                    v-for="(section, index) in sections"
                    :key="section.id"
                    :description="section.content.body"
                    :title="section.content.title"
                    :image="section.content.image"
                    :image-alt-text="section.content.image_alt_text"
                    :is-reversed="index % 2 != 0"
                    :show-dots="true"
            />

            <EditionSlider :editions="editions"/>

            <section class="py-16 lg:py-32 relative overflow-hidden">
                <g-image
                        class="absolute z-0"
                        role="presentation"
                        alt
                        src="../images/dots_big.svg"
                        :style="{ height: '600px', left: '80px', top: '60px' }"
                />

                <div class="z-20 max-w-xl mx-auto relative">
                    <Title class="text-lg md:text-xl lg:text-4xl">DevTreff Archiv</Title>
                </div>

                <ImageSlider class="relative z-10">
                    <BlogTeaserImage
                            v-for="event in pastEventsWithBlogPosts"
                            :key="event.id"
                            target="_blank"
                            :href="event.content.blog_post.full_slug"
                            :src="event.content.teaser_image"
                            :title="event.content.location.content.city"
                            :subtitle="formatDate(event.content.date)"
                    />
                </ImageSlider>
            </section>
        </template>
    </FullImageLayout>
</template>

<script>
    import Button from "~/components/Button.vue";
    import EditionSlider from "~/components/EditionSlider.vue";
    import Title from "~/components/Title.vue";
    import ImageSlider from "~/components/ImageSlider.vue";
    import {DateTime} from "luxon";
    import BlogTeaserImage from "~/components/BlogTeaserImage.vue";
    import formatDate from "~/helpers/formatDate.ts";

    export default {
        components: {
            Button,
            Title,
            ImageSlider,
            EditionSlider,
            BlogTeaserImage
        },
        metaInfo: {
            title: "DevTreff"
        },
        computed: {
            sections() {
                return this.$static.allMainSection.edges.map(({node}) => node);
            },
            editions() {
                return this.$static.allEdition.edges.map(({node}) => node);
            },
            mappedEvents() {
                return this.$static.allEvent.edges.map(({node}) => {
                    const luxonDate = DateTime.fromFormat(
                            node.content.date,
                            "yyyy-MM-dd HH:mm"
                    );
                    return {
                        ...node,
                        luxonDate
                    };
                });
            },
            pastEvents() {
                return this.mappedEvents.filter(({luxonDate}) => {
                    const now = DateTime.local();
                    return luxonDate < now;
                });
            },
            pastEventsWithBlogPosts() {
                return this.pastEvents
                .filter(({content}) => content.blog_post)
                .reverse();
            },
            upcomingEvents() {
                let upcoming = this.mappedEvents.filter(({luxonDate}) => {
                    const now = DateTime.local();
                    return luxonDate >= now;
                });

                if (upcoming.length === 0) {
                    upcoming = this.mappedEvents
                    .slice()
                    .sort((a, b) => b.luxonDate - a.luxonDate)
                    .slice(0, 1);
                }
                return upcoming;
            }
        },
        methods: {
            formatDate
        }
    };
</script>
