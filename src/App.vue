<script setup>
import {onBeforeUnmount, onMounted, ref, computed} from 'vue';
import gsap from 'gsap';
import ScrollTrigger from 'gsap/ScrollTrigger';
import ScrollToPlugin from 'gsap/ScrollToPlugin';
import {SplitText} from "gsap/SplitText";
import {throttle} from 'lodash-es';

import ArielaLogo from './assets/ariela-logo.svg?component';
import SportFiveLogo from './assets/sport5-logo.svg?component';
import AthenaLogo from './assets/athena-logo.svg?component';
import SportsMinistryLogo from './assets/sports-ministry-logo.svg?component';
import WingateLogo from './assets/wingate-logo.svg?component';

gsap.registerPlugin(ScrollTrigger);
gsap.registerPlugin(SplitText);
gsap.registerPlugin(ScrollToPlugin)


const logoRef = ref(null);
const navArielaRef = ref(null);
const navBuyTicketsRef = ref(null);
const firstContentTitleRef = ref(null);
const firstContentDescRef = ref(null);
const backgroundVideoRef = ref(null);
const clearVideoRef = ref(null)
const maskStyle = ref({})
const menuItemsRef = ref([]);
const contentItemsRef = ref([]);
const splitTextRef = ref([]);
const linesRef = ref([]);
const playingVideo = ref('intro');
const aboutRef = ref(null);
const firstDayRef = ref(null);
const secondDayRef = ref(null);
const showBoycottModal = ref(false);
const showBookPages = ref(false)
const footerRef = ref(null);
const boycottImages = [
    '/assets/boycott-images/1.webp', '/assets/boycott-images/2.avif', '/assets/boycott-images/3.webp',
    '/assets/boycott-images/4.png', '/assets/boycott-images/5.webp', '/assets/boycott-images/6.avif',
    '/assets/boycott-images/7.avif', '/assets/boycott-images/8.webp', '/assets/boycott-images/9.avif',
    '/assets/boycott-images/10.webp', '/assets/boycott-images/11.webp', '/assets/boycott-images/12.jpg',
    '/assets/boycott-images/13.jpg', '/assets/boycott-images/14.jpg', '/assets/boycott-images/15.jpg',
    '/assets/boycott-images/16.webp', '/assets/boycott-images/17.webp', '/assets/boycott-images/18.png',
    '/assets/boycott-images/19.png', '/assets/boycott-images/20.jpg', '/assets/boycott-images/21.png',
    '/assets/boycott-images/22.png'
];
const currentIndex = ref(0);
const currentImage = computed(() => boycottImages[currentIndex.value]);
const boycottContainerRef = ref(null);
const boycottImageRef = ref(null);
let selectedItemID = ref(null);
const currentBookPage = ref(0);
const bookPagesImages = [
    '/assets/book-pages/1.png', '/assets/book-pages/2.png', '/assets/book-pages/3.png',
    '/assets/book-pages/4.png', '/assets/book-pages/5.png', '/assets/book-pages/6.png',
    '/assets/book-pages/7.png', '/assets/book-pages/8.png', '/assets/book-pages/9.png',
    '/assets/book-pages/10.png', '/assets/book-pages/11.png', '/assets/book-pages/12.png',
    '/assets/book-pages/13.png', '/assets/book-pages/14.png', '/assets/book-pages/15.png',
    '/assets/book-pages/16.png'
];

const getDistanceToElements = (x, y) => {
    // Get all elements with the exclusion-element class
    const exclusionElements = document.querySelectorAll('.exclusion-element')
    let minDistance = Infinity

    exclusionElements.forEach(element => {
        const rect = element.getBoundingClientRect()

        // Check if point is inside element
        if (x >= rect.left && x <= rect.right &&
            y >= rect.top && y <= rect.bottom) {
            minDistance = 0 // Inside element
            return
        }

        // Calculate distance to nearest edge of element
        const dx = Math.max(rect.left - x, 0, x - rect.right)
        const dy = Math.max(rect.top - y, 0, y - rect.bottom)
        const distance = Math.sqrt(dx * dx + dy * dy)

        minDistance = Math.min(minDistance, distance)
    })

    return minDistance
}

const updateMask = (e) => {
    requestAnimationFrame(() => {
        const x = e.clientX
        const y = e.clientY
        const centerRadius = 25
        const fadeRadius = 150

        const distanceToElement = getDistanceToElements(x, y)
        const fadeDistance = 2

        let opacity = 1
        if (distanceToElement < fadeDistance) {
            opacity = Math.max(0.1, distanceToElement / fadeDistance)
        }

        maskStyle.value = {
            WebkitMask: `radial-gradient(circle at ${x}px ${y}px,
                rgba(255,255,255,${opacity}) ${centerRadius}px,
                transparent ${fadeRadius}px)`,
            mask: `radial-gradient(circle at ${x}px ${y}px,
                rgba(255,255,255,${opacity}) ${centerRadius}px,
                transparent ${fadeRadius}px)`,
            transition: 'mask 0.1s cubic-bezier(.16,1,.91,.37), -webkit-mask 0.1s cubic-bezier(.16,1,.91,.37)'
        }
    })
}

const throttledUpdateMask = throttle(updateMask, 8)

const handleKeyDown = (e) => {
    if (!showBookPages.value) return;

    if (e.key === 'Escape') {
        closeBookModal();
    } else if (e.key === 'ArrowRight') {
        nextBookPage();
    } else if (e.key === 'ArrowLeft') {
        prevBookPage();
    }
};

onMounted(() => {
    document.addEventListener('mousemove', throttledUpdateMask, {passive: true});
    document.addEventListener('keydown', handleKeyDown);

    // Set initial video sources
    if (clearVideoRef.value) {
        clearVideoRef.value.src = '/assets/background-videos/intro.mp4';
        clearVideoRef.value.load();
        clearVideoRef.value.play();
    }
    if (backgroundVideoRef.value) {
        backgroundVideoRef.value.src = '/assets/background-videos/intro.mp4';
        backgroundVideoRef.value.load();
        backgroundVideoRef.value.play();
    }

    // Initialize refs for menu items and content sections
    menuItemsRef.value = document.querySelectorAll('.content__menu__item');
    contentItemsRef.value = document.querySelectorAll('.content__main__item');

    // Set initial positions for menu items
    gsap.set(menuItemsRef.value[1], {y: 200, opacity: 0});
    gsap.set(menuItemsRef.value[2], {y: 400, opacity: 0});

    // Set initial positions for content items
    gsap.set(contentItemsRef.value[0], {opacity: 1, y: 0});
    gsap.set(contentItemsRef.value[1], {opacity: 0, y: 100});
    gsap.set(contentItemsRef.value[2], {opacity: 0, y: 100});

    // Set initial position for footer
    gsap.set(footerRef.value, {opacity: 0, y: 100});

    // Set initial positions for descriptions
    const descriptions = document.querySelectorAll('.content__main__item__description');
    gsap.set(descriptions, {opacity: 0, scale: 0.95});

    // Set initial positions for slots
    const firstContentSlots = document.querySelectorAll('.content__main__item:nth-child(1) .content__main__item__slot');
    const secondContentSlots = document.querySelectorAll('.content__main__item:nth-child(2) .content__main__item__slot');
    const thirdContentSlots = document.querySelectorAll('.content__main__item:nth-child(3) .content__main__item__slot');

    gsap.set(firstContentSlots, {y: 1000, opacity: 0});
    gsap.set(secondContentSlots, {y: 50, opacity: 0, scale: 0.95, filter: 'blur(10px)'});
    gsap.set(thirdContentSlots, {y: 50, opacity: 0, scale: 0.95, filter: 'blur(10px)'});

    // Create the scroll-triggered animation
    let tl = gsap.timeline({
        scrollTrigger: {
            trigger: '.content',
            start: 'top top',
            end: '+=3000vh', // Increased from 800vh to make pin last much longer
            scrub: 3,
            pin: true,
            anticipatePin: 1,
        }
    });

    // Add logo animation to separate timeline that starts earlier
    const logo = logoRef.value;
    const nav = document.querySelector('.navbar');
    const navRect = nav.getBoundingClientRect();
    const navCenterY = navRect.top + navRect.height / 2;

    // Create separate timeline for logo animation
    gsap.timeline({
        scrollTrigger: {
            trigger: '.content-wrapper',
            start: 'top bottom',
            end: 'top+=100vh bottom',
            scrub: 1,
        }
    })
        .to(logo, {
            top: `${navCenterY}px`,
            scale: 1,
            duration: 2,
            ease: 'power2.inOut'
        });

    // PHASE 1: About section appears
    tl.to(descriptions[0], {
        opacity: 1,
        scale: 1,
        duration: 2,
        ease: 'power2.inOut'
    }, 0)
        .to({}, {duration: 8}) // Increased hold duration from 3 to 8

        // PHASE 2: Second menu item + content transition + all slots appear
        .to(menuItemsRef.value[1], {
            y: 0,
            opacity: 1,
            duration: 2,
            ease: 'power3.out'
        })
        .to(contentItemsRef.value[0], {
            opacity: 0,
            y: -100,
            duration: 2,
            ease: 'power3.out'
        }, '<')
        .to(contentItemsRef.value[1], {
            opacity: 1,
            y: 0,
            duration: 2,
            ease: 'power3.out'
        }, '<0.5')
        .to(descriptions[0], {
            opacity: 0,
            scale: 0.95,
            duration: 1.2,
            ease: 'power3.out'
        }, '<')
        .to(descriptions[1], {
            opacity: 1,
            scale: 1,
            duration: 2,
            ease: 'power3.out'
        }, '<0.5')
        // All second section slots appear together
        .to(secondContentSlots, {
            y: 0,
            opacity: 1,
            scale: 1,
            filter: 'blur(0px)',
            duration: 1.8,
            stagger: 0.2,
            ease: 'power3.out'
        }, '<1') // Start shortly after content transition begins
        .to({}, {duration: 8}) // Increased hold duration from 3 to 8

        // PHASE 3: Third menu item + content transition + all slots appear
        .to(menuItemsRef.value[2], {
            y: 0,
            opacity: 1,
            duration: 2,
            ease: 'power3.out'
        })
        .to(contentItemsRef.value[1], {
            opacity: 0,
            y: -100,
            duration: 2,
            ease: 'power3.out'
        }, '<')
        .to(contentItemsRef.value[2], {
            opacity: 1,
            y: 0,
            duration: 2,
            ease: 'power3.out'
        }, '<0.5')
        .to(descriptions[1], {
            opacity: 0,
            scale: 0.95,
            duration: 1.2,
            ease: 'power3.out'
        }, '<')
        .to(descriptions[2], {
            opacity: 1,
            scale: 1,
            duration: 2,
            ease: 'power3.out'
        }, '<0.5')
        // All third section slots appear together
        .to(thirdContentSlots, {
            y: 0,
            opacity: 1,
            scale: 1,
            filter: 'blur(0px)',
            duration: 1.8,
            stagger: 0.2,
            ease: 'power3.out'
        }, '<1')
        .to({}, {duration: 8}) // Increased hold duration from 3 to 8

        // // Final exit animation
        // .to(contentItemsRef.value[2], {
        //     opacity: 0,
        //     y: -100,
        //     duration: 2,
        //     ease: 'power3.out'
        // })
        // .to(descriptions[2], {
        //     opacity: 0,
        //     scale: 0.95,
        //     duration: 1.2,
        //     ease: 'power3.out'
        // }, '<')
        // Add footer animation
        .to(footerRef.value, {
            opacity: 1,
            y: 0,
            duration: 2,
            ease: 'power3.out'
        }, '<0.5');

    // Original animations for video blur only (logo is now in main timeline)
    // Note: We animate video blur and overlay separately to avoid the filter
    // property affecting the overlay and causing a black screen
    const video = backgroundVideoRef.value;
    if (video) {
        // Video blur happens after logo animation
        gsap.to(video, {
            scrollTrigger: {
                trigger: '.content-wrapper',
                start: 'top+=100vh bottom',
                end: 'top+=200vh bottom',
                scrub: 1,
            },
            filter: 'blur(45px)',
            ease: 'power3.out',
            willChange: 'filter',
            force3D: true
        });

        // Separate animation for overlay
        gsap.to('.video-overlay', {
            scrollTrigger: {
                trigger: '.content-wrapper',
                start: 'top+=100vh bottom',
                end: 'top+=200vh bottom',
                scrub: 1,
            },
            opacity: 0.5,
            ease: 'power3.out'
        });

        // Add scroll trigger for about text to reset video
        gsap.to(video, {
            scrollTrigger: {
                trigger: '.content__main__item__description',
                start: 'top center',
                end: 'bottom center',
                onEnter: () => {
                    if (selectedItemID.value === null) {
                        playingVideo.value = 'intro';
                        if (clearVideoRef.value) {
                            clearVideoRef.value.src = '/assets/background-videos/intro.mp4';
                            clearVideoRef.value.load();
                            clearVideoRef.value.play();
                        }
                        if (backgroundVideoRef.value) {
                            backgroundVideoRef.value.src = '/assets/background-videos/intro.mp4';
                            backgroundVideoRef.value.load();
                            backgroundVideoRef.value.play();
                        }
                    }
                }
            }
        });
    }

    if (backgroundVideoRef.value && clearVideoRef.value) {
        backgroundVideoRef.value.addEventListener('timeupdate', () => {
            if (clearVideoRef.value) {
                clearVideoRef.value.currentTime = backgroundVideoRef.value.currentTime
            }
        })
    }

    // First content item animation
    const title = firstContentTitleRef.value;
    const desc = firstContentDescRef.value;

    if (title && desc) {
        // Set initial state
        gsap.set(title, {x: -100, opacity: 0});
        gsap.set(desc, {opacity: 0});

        // Create timeline for title
        gsap.to(title, {
            scrollTrigger: {
                trigger: title,
                start: 'top bottom-=100',
                end: 'top center',
                scrub: 1,
            },
            x: 0,
            opacity: 1,
            ease: 'power2.out'
        });

        const split = SplitText.create(desc, {type: 'lines', mask: 'lines', linesClass: 'line++'})
        splitTextRef.value.push(split);

        linesRef.value.push(...split.lines)


        // Create timeline for description
        gsap.to(desc, {
            scrollTrigger: {
                trigger: title,
                start: 'top center',
                end: 'top top+=100',
                scrub: 1,
            },
            opacity: 1,
            ease: 'power2.out'
        });
    }
});

onBeforeUnmount(() => {
    ScrollTrigger.getAll().forEach(t => t.kill());
    document.removeEventListener('mousemove', updateMask);
    document.removeEventListener('keydown', handleKeyDown);
});

const clickScheduleItem = (itemID) => {
    console.log('clickScheduleItem', itemID);
    if (selectedItemID.value === itemID) {
        selectedItemID.value = null; // Deselect if already selected
        // Apply blur to video only
        if (backgroundVideoRef.value) {
            gsap.to(backgroundVideoRef.value, {
                filter: 'blur(45px)',
                duration: 0.3,
                ease: 'power2.out'
            });
            gsap.to('.video-overlay', {
                opacity: 0.5,
                duration: 0.3,
                ease: 'power2.out'
            });
        }
    } else {
        selectedItemID.value = itemID; // Select the clicked item
        // Apply blur to video only
        if (backgroundVideoRef.value) {
            gsap.to(backgroundVideoRef.value, {
                filter: 'blur(45px)',
                duration: 0.3,
                ease: 'power2.out'
            });
            gsap.to('.video-overlay', {
                opacity: 0.5,
                duration: 0.3,
                ease: 'power2.out'
            });
        }
    }
}

const mouseOverScheduleItem = async (itemID) => {
    // Check if the hovered item is within the active section
    const activeSection = document.querySelector(`.content__main__item[style*="opacity: 1"]`);
    const hoveredItem = document.querySelector(`[data-item-id="${itemID}"]`);
    if (!activeSection || !hoveredItem || !activeSection.contains(hoveredItem)) return;

    if (itemID === 'about' || itemID === "8" || itemID === 8) {
        itemID = 'intro';
    }
    console.log('mouseOverScheduleItem', itemID, 'video playing:', playingVideo.value);
    if (playingVideo.value !== itemID) {
        playingVideo.value = itemID;
        const videoPath = `/assets/background-videos/${itemID}.mp4`;

        try {
            if (clearVideoRef.value) {
                clearVideoRef.value.src = videoPath;
                await clearVideoRef.value.load();
                await clearVideoRef.value.play();
            }
            if (backgroundVideoRef.value) {
                backgroundVideoRef.value.src = videoPath;
                await backgroundVideoRef.value.load();
                await backgroundVideoRef.value.play();
                // Only remove blur if no slot is selected
                if (selectedItemID.value === null) {
                    gsap.to(backgroundVideoRef.value, {
                        filter: 'blur(0px)',
                        duration: 0.8,
                        ease: 'power2.inOut'
                    });
                    gsap.to('.video-overlay', {
                        opacity: 0,
                        duration: 0.8,
                        ease: 'power2.inOut'
                    });
                }
            }
        } catch (error) {
            console.error('Error loading video:', error);
            // Fallback to intro video on error
            playingVideo.value = 'intro';
            if (clearVideoRef.value) {
                clearVideoRef.value.src = '/assets/background-videos/intro.mp4';
                await clearVideoRef.value.load();
                await clearVideoRef.value.play();
            }
            if (backgroundVideoRef.value) {
                backgroundVideoRef.value.src = '/assets/background-videos/intro.mp4';
                await backgroundVideoRef.value.load();
                await backgroundVideoRef.value.play();
            }
        }
    } else {
        // If it's the same video, only remove blur if no slot is selected
        if (backgroundVideoRef.value && selectedItemID.value === null) {
            gsap.to(backgroundVideoRef.value, {
                filter: 'blur(0px)',
                duration: 0.8,
                ease: 'power2.inOut'
            });
            gsap.to('.video-overlay', {
                opacity: 0,
                duration: 0.8,
                ease: 'power2.inOut'
            });
        }
    }
}

const mouseLeaveScheduleItem = async (itemID) => {
    // Check if the hovered item is within the active section
    const activeSection = document.querySelector(`.content__main__item[style*="opacity: 1"]`);
    const hoveredItem = document.querySelector(`[data-item-id="${itemID}"]`);
    if (!activeSection || !hoveredItem || !activeSection.contains(hoveredItem)) return;

    // Keep current video playing but apply blur if no slot is selected
    console.log('mouseLeaveScheduleItem', itemID, 'video playing:', playingVideo.value);
    if (backgroundVideoRef.value && selectedItemID.value === null) {
        console.log('Adding blur to background video');
        gsap.to(backgroundVideoRef.value, {
            filter: 'blur(45px)',
            duration: 0.8,
            ease: 'power2.inOut'
        });
        gsap.to('.video-overlay', {
            opacity: 0.5,
            duration: 0.8,
            ease: 'power2.inOut'
        });
    }
}

const onBoycottMouseMove = throttle((e) => {
    const {left, width} = boycottContainerRef.value.getBoundingClientRect();
    let pct = (e.clientX - left) / width;
    pct = Math.min(1, Math.max(0, pct));
    const idx = Math.floor(pct * boycottImages.length);

    if (idx !== currentIndex.value) {
        currentIndex.value = idx;
        gsap.fromTo(
            boycottImageRef.value,
            {opacity: 0},
            {opacity: 1, duration: 0.3, ease: 'power1.out'}
        );
    }
}, 100); // 300ms throttle

const returnToIntroVideo = () => {
    if (playingVideo.value !== 'intro') {
        playingVideo.value = 'intro';
        if (clearVideoRef.value) {
            clearVideoRef.value.src = '/assets/background-videos/intro.mp4';
            clearVideoRef.value.load();
            clearVideoRef.value.play();
        }
        if (backgroundVideoRef.value) {
            backgroundVideoRef.value.src = '/assets/background-videos/intro.mp4';
            backgroundVideoRef.value.load();
            backgroundVideoRef.value.play();
            gsap.to(backgroundVideoRef.value, {
                filter: 'blur(45px)',
                duration: 0.8,
                ease: 'power2.inOut'
            });
            gsap.to('.video-overlay', {
                opacity: 0.5,
                duration: 0.8,
                ease: 'power2.inOut'
            });
        }
    }
}

const nextBookPage = () => {
    currentBookPage.value = (currentBookPage.value + 1) % bookPagesImages.length;
};

const prevBookPage = () => {
    if (currentBookPage.value > 0) {
        currentBookPage.value--;
    }
};

const closeBookModal = () => {
    showBookPages.value = false;
    currentBookPage.value = 0;
};

const menuClick = (section) => {
    console.log('menuClick', section);
    if (section === 'about') {
        gsap.to(window, {duration: 2, scrollTo: "#about"})
    } else if (section === 'first_day') {
        gsap.to(window, {duration: 2, scrollTo: "#first_day"})
    } else if (section === 'second_day') {
        gsap.to(window, {duration: 2, scrollTo: "#second_day"})
    }
};

</script>

<template>
    <div>
        <div class="navbar">
            <div class="navbar__ariela" ref="navArielaRef">
                <ArielaLogo></ArielaLogo>
            </div>
            <div class="navbar__logo" ref="logoRef">
                <div class="navbar__logo__top">
                    <div class="navbar__logo__top__text">Free Kick</div>
                    <div class="navbar__logo__top__text">בעיטה חופשית</div>
                </div>
                <div class="navbar__logo__bottom">
                    <div class="navbar__logo__bottom__text">29.5.25</div>
                    <div class="navbar__logo__bottom__arrow">
                        <svg width="100" height="9" viewBox="0 0 100 9" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <path d="M4.64641 8.99989L4.64641 5.24362L100 5.24374L99.9979 3.52375L4.64641 3.52363L4.64641 -0.000125178L0.00421866 4.49988L4.64641 8.99989Z"/>
                        </svg>
                    </div>
                    <div class="navbar__logo__bottom__text">30.5.25</div>
                </div>
            </div>
            <div class="navbar__buy_tickets" ref="navBuyTicketsRef">
                <div class="spinning-circle">
                    <svg viewBox="0 0 100 100" class="spinning-circle__svg">
                        <defs>
                            <path id="circlePath" d="M50,50 m-35,0 a35,35 0 1,1 70,0 a35,35 0 1,1 -70,0"/>
                        </defs>
                        <text fill="currentColor">
                            <textPath xlink:href="#circlePath" startOffset="0%">
                                כרטיסים • כרטיסים •
                            </textPath>
                        </text>
                    </svg>
                </div>
            </div>
        </div>
        <div class="boycott_modal" v-if="showBoycottModal" @keydown.esc="showBoycottModal = false" @click.self="showBoycottModal = false">
            <div class="boycott_modal__content">
                <div class="boycott_modal__content__right" ref="boycottContainerRef" @mousemove="onBoycottMouseMove">
                    <img
                        :src="currentImage"
                        alt="Boycott history"
                        class="boycott_modal__content__right__image"
                        ref="boycottImageRef"
                    />
                </div>
                <div class="boycott_modal__content__left">
                    <div class="boycott_modal__content__left__title">״כדורגל הוא משחק שאינו מתאים לנשים ואין לעודדן לשחק בו.״</div>
                    <div class="boycott_modal__content__left__text">
                        בשנת 1921, התאחדות הכדורגל האנגלית (The FA) אסרה על קבוצות נשים לשחק במגרשים של ההתאחדות, בטענה כי "הכדורגל אינו מתאים לנשים ואינו ראוי שיאומץ על ידיהן". האיסור לא נבע רק משיקולים רפואיים או מגדריים, אלא גם מתחרות כלכלית – משחקי נשים זכו לפופולריות רבה וגייסו כספים לצדקה, לעיתים אף יותר ממשחקי הגברים. כתוצאה מהחרם, נדחק כדורגל הנשים אל השוליים, והתקשה להתקיים ללא תשתיות, הכרה או משאבים. ב-1975 הוסר האיסור באופן רשמי.
                    </div>
                </div>
            </div>
        </div>
        <div class="book_pages" v-if="showBookPages" @keydown.esc="closeBookModal" @click.self="closeBookModal">
            <div class="book_pages__content">
                <img
                    :src="bookPagesImages[currentBookPage]"
                    alt="Book page"
                    class="book_pages__content__image"
                    @click="nextBookPage"
                />
            </div>
        </div>
        <div class="content-wrapper">
            <!--            <div class="content-spacer"></div>-->
            <div class="content" v-show="!showBoycottModal && !showBookPages">
                <div class="content__menu">
                    <div class="content__menu__item" id="about" @click="menuClick('about')">אודות</div>
                    <div class="content__menu__item" id="first_day" @click="menuClick('first_day')">יום 1 29.5</div>
                    <div class="content__menu__item" id="second_day" @click="menuClick('second_day')">יום 2 30.5</div>
                </div>
                <div class="content__main">
                    <div class="content__main__item exclusion-element" id="about" ref="aboutRef" @mouseenter="mouseOverScheduleItem('about')" @mouseleave="mouseLeaveScheduleItem('about')">
                        <div class="content__main__item__description" @mouseenter="returnToIntroVideo">
                            לכבוד 50 שנה לביטול <span class="highlight-content-text" @click="showBoycottModal = true">החרם על כדורגל הנשים,</span> יתקיים בתל אביב כנס מחקרי בן יומיים שייפתח צוהר לעולמו המורכב של כדורגל הנשים בארץ ובעולם. נבחן את התחום כתופעה חברתית ותרבותית, באמצעות הרצאות, פאנלים, ושיחות עם חוקרות מהארץ ומהעולם. בנוסף, תושק <span class="highlight-content-text" @click="showBookPages = true">חוברת מחקרית</span> חדשה העוסקת בהיסטוריה ובתרבות הכדורגל. זהו מפגש רב-תחומי שמציג את כדורגל הנשים לא רק כספורט, אלא גם כמרחב של תשוקה, התנגדות ושינוי חברתי.
                        </div>
                    </div>
                    <div class="content__main__item first_day" id="first_day" ref="firstDayRef">
                        <div class="content__main__item__slot" data-item-id="1" @click="clickScheduleItem(1)" @mouseenter="mouseOverScheduleItem(1)" @mouseleave="mouseLeaveScheduleItem(1)">
                            <div class="content__main__item__slot__wrapper exclusion-element">
                                <div class="content__main__item__slot__time"><span>[</span>9:00<span>]</span></div>
                                <div class="content__main__item__slot__info">
                                    <div class="content__main__item__slot__info__speakers">תמר רפפורט</div>
                                    <div class="content__main__item__slot__info__title">שריקת פתיחה</div>
                                </div>
                            </div>
                            <div class="content__main__item__slot__description exclusion-element" v-if="selectedItemID === 1">
                                איך נאסר על נשים לשחק כדורגל במשך עשרות שנים – ואיך הן הצליחו להחזיר לעצמן את המגרש. מבט היסטורי על מקורות החרם הבריטי, השפעתו המקומית, ופריצת הדרך שהובילה למהפכה מגדרית על הדשא. נצלול לתוך השיח הפוליטי, החברתי והתרבותי של אותה תקופה, ונשאל – כיצד השפיע החרם, והאם השפעה זו מורגשת עד היום?
                            </div>
                        </div>
                        <div class="content__main__item__slot" data-item-id="2" @click="clickScheduleItem(2)" @mouseenter="mouseOverScheduleItem(2)" @mouseleave="mouseLeaveScheduleItem(2)">
                            <div class="content__main__item__slot__wrapper exclusion-element">
                                <div class="content__main__item__slot__time"><span>[</span>11:00<span>]</span></div>
                                <div class="content__main__item__slot__info">
                                    <div class="content__main__item__slot__info__speakers">קרן שפילד</div>
                                    <div class="content__main__item__slot__info__title">הרכב ראשון</div>
                                </div>
                            </div>
                            <div class="content__main__item__slot__description exclusion-element" v-if="selectedItemID === 2">
                                איך נאסר על נשים לשחק כדורגל במשך עשרות שנים – ואיך הן הצליחו להחזיר לעצמן את המגרש. מבט היסטורי על מקורות החרם הבריטי, השפעתו המקומית, ופריצת הדרך שהובילה למהפכה מגדרית על הדשא. נצלול לתוך השיח הפוליטי, החברתי והתרבותי של אותה תקופה, ונשאל – כיצד השפיע החרם, והאם השפעה זו מורגשת עד היום?
                            </div>
                        </div>
                        <div class="content__main__item__slot" data-item-id="3" @click="clickScheduleItem(3)" @mouseenter="mouseOverScheduleItem(3)" @mouseleave="mouseLeaveScheduleItem(3)">
                            <div class="content__main__item__slot__wrapper exclusion-element">
                                <div class="content__main__item__slot__time"><span>[</span>14:00<span>]</span></div>
                                <div class="content__main__item__slot__info">
                                    <div class="content__main__item__slot__info__speakers">ד״ר גייל ניושאם</div>
                                    <div class="content__main__item__slot__info__title">משחק חוץ</div>
                                </div>
                            </div>
                            <div class="content__main__item__slot__description exclusion-element" v-if="selectedItemID === 3">
                                איך נאסר על נשים לשחק כדורגל במשך עשרות שנים – ואיך הן הצליחו להחזיר לעצמן את המגרש. מבט היסטורי על מקורות החרם הבריטי, השפעתו המקומית, ופריצת הדרך שהובילה למהפכה מגדרית על הדשא. נצלול לתוך השיח הפוליטי, החברתי והתרבותי של אותה תקופה, ונשאל – כיצד השפיע החרם, והאם השפעה זו מורגשת עד היום?
                            </div>
                        </div>
                        <div class="content__main__item__slot" data-item-id="4" @click="clickScheduleItem(4)" @mouseenter="mouseOverScheduleItem(4)" @mouseleave="mouseLeaveScheduleItem(4)">
                            <div class="content__main__item__slot__wrapper exclusion-element">
                                <div class="content__main__item__slot__time"><span>[</span>16:00<span>]</span></div>
                                <div class="content__main__item__slot__info">
                                    <div class="content__main__item__slot__info__speakers">ד״ר תהילה שרעבי</div>
                                    <div class="content__main__item__slot__info__title">הגביע העולמי</div>
                                </div>
                            </div>
                            <div class="content__main__item__slot__description exclusion-element" v-if="selectedItemID === 4">
                                איך נאסר על נשים לשחק כדורגל במשך עשרות שנים – ואיך הן הצליחו להחזיר לעצמן את המגרש. מבט היסטורי על מקורות החרם הבריטי, השפעתו המקומית, ופריצת הדרך שהובילה למהפכה מגדרית על הדשא. נצלול לתוך השיח הפוליטי, החברתי והתרבותי של אותה תקופה, ונשאל – כיצד השפיע החרם, והאם השפעה זו מורגשת עד היום?
                            </div>
                        </div>
                    </div>
                    <div class="content__main__item second_day" id="second_day" ref="secondDayRef">
                        <div class="content__main__item__slot" data-item-id="5" @click="clickScheduleItem(5)" @mouseenter="mouseOverScheduleItem(5)" @mouseleave="mouseLeaveScheduleItem(5)">
                            <div class="content__main__item__slot__wrapper exclusion-element">
                                <div class="content__main__item__slot__time"><span>[</span>9:00<span>]</span></div>
                                <div class="content__main__item__slot__info">
                                    <div class="content__main__item__slot__info__speakers">ד״ר נויה וימלט</div>
                                    <div class="content__main__item__slot__info__title">שידור ישיר</div>
                                </div>
                            </div>
                            <div class="content__main__item__slot__description exclusion-element" v-if="selectedItemID === 5">
                                איך נאסר על נשים לשחק כדורגל במשך עשרות שנים – ואיך הן הצליחו להחזיר לעצמן את המגרש. מבט היסטורי על מקורות החרם הבריטי, השפעתו המקומית, ופריצת הדרך שהובילה למהפכה מגדרית על הדשא. נצלול לתוך השיח הפוליטי, החברתי והתרבותי של אותה תקופה, ונשאל – כיצד השפיע החרם, והאם השפעה זו מורגשת עד היום?
                            </div>
                        </div>
                        <div class="content__main__item__slot" data-item-id="6" @click="clickScheduleItem(6)" @mouseenter="mouseOverScheduleItem(6)" @mouseleave="mouseLeaveScheduleItem(6)">
                            <div class="content__main__item__slot__wrapper exclusion-element">
                                <div class="content__main__item__slot__time"><span>[</span>11:00<span>]</span></div>
                                <div class="content__main__item__slot__info">
                                    <div class="content__main__item__slot__info__speakers">ספיר ברמן</div>
                                    <div class="content__main__item__slot__info__title">שריקת השופטת</div>
                                </div>
                            </div>
                            <div class="content__main__item__slot__description exclusion-element" v-if="selectedItemID === 6">
                                איך נאסר על נשים לשחק כדורגל במשך עשרות שנים – ואיך הן הצליחו להחזיר לעצמן את המגרש. מבט היסטורי על מקורות החרם הבריטי, השפעתו המקומית, ופריצת הדרך שהובילה למהפכה מגדרית על הדשא. נצלול לתוך השיח הפוליטי, החברתי והתרבותי של אותה תקופה, ונשאל – כיצד השפיע החרם, והאם השפעה זו מורגשת עד היום?
                            </div>
                        </div>
                        <div class="content__main__item__slot" data-item-id="7" @click="clickScheduleItem(7)" @mouseenter="mouseOverScheduleItem(7)" @mouseleave="mouseLeaveScheduleItem(7)">
                            <div class="content__main__item__slot__wrapper exclusion-element">
                                <div class="content__main__item__slot__time"><span>[</span>14:00<span>]</span></div>
                                <div class="content__main__item__slot__info">
                                    <div class="content__main__item__slot__info__speakers">מורן אזולאי • ד״ר טל כוכבי</div>
                                    <div class="content__main__item__slot__info__title">מלכת הרחבה</div>
                                </div>
                            </div>
                            <div class="content__main__item__slot__description exclusion-element" v-if="selectedItemID === 7">
                                איך נאסר על נשים לשחק כדורגל במשך עשרות שנים – ואיך הן הצליחו להחזיר לעצמן את המגרש. מבט היסטורי על מקורות החרם הבריטי, השפעתו המקומית, ופריצת הדרך שהובילה למהפכה מגדרית על הדשא. נצלול לתוך השיח הפוליטי, החברתי והתרבותי של אותה תקופה, ונשאל – כיצד השפיע החרם, והאם השפעה זו מורגשת עד היום?
                            </div>
                        </div>
                        <div class="content__main__item__slot" data-item-id="8" @click="clickScheduleItem(8)" @mouseenter="mouseOverScheduleItem(8)" @mouseleave="mouseLeaveScheduleItem(8)">
                            <div class="content__main__item__slot__wrapper exclusion-element">
                                <div class="content__main__item__slot__time"><span>[</span>16:00<span>]</span></div>
                                <div class="content__main__item__slot__info">
                                    <div class="content__main__item__slot__info__speakers">ענת יאיר</div>
                                    <div class="content__main__item__slot__info__title">״כדורגל: החוברת״</div>
                                </div>
                            </div>
                            <div class="content__main__item__slot__description exclusion-element" v-if="selectedItemID === 8">
                                איך נאסר על נשים לשחק כדורגל במשך עשרות שנים – ואיך הן הצליחו להחזיר לעצמן את המגרש. מבט היסטורי על מקורות החרם הבריטי, השפעתו המקומית, ופריצת הדרך שהובילה למהפכה מגדרית על הדשא. נצלול לתוך השיח הפוליטי, החברתי והתרבותי של אותה תקופה, ונשאל – כיצד השפיע החרם, והאם השפעה זו מורגשת עד היום?
                                <div class="content__main__item__slot__description__button" @click="showBookPages = true">
                                    להצצה בחוברת
                                    <svg width="32" height="13" viewBox="0 0 32 13" fill="none" xmlns="http://www.w3.org/2000/svg">
                                        <path d="M-2.72457e-07 6.23321L7.31712 0.835186L7.31712 11.6312L-2.72457e-07 6.23321Z" fill="white"/>
                                        <rect x="7" y="5" width="25" height="2" fill="white"/>
                                    </svg>

                                </div>
                            </div>
                        </div>
                    </div>
                </div>

            </div>
        </div>
        <div class="footer" ref="footerRef">
            <div class="footer__content">
                <div class="footer__content__line">
                    בית אריאלה, שדרות שאול המלך 25, תל אביב
                </div>
                <div class="footer__content__line">
                    ליצירת קשר freekick@gmail.com | טל׳ 03-8299020
                </div>
                <div class="footer__content__line">
                    <div class="footer__content__link__logo">
                        <img src="./assets/footer-logos/sport5.png" alt="sport5 logo">
                    </div>
                    <div class="footer__content__link__logo">
                        <img src="./assets/footer-logos/athena.png" alt="athena logo">
                    </div>
                    <div class="footer__content__link__logo">
                        <img src="./assets/footer-logos/ministry.png" alt="ministry logo">
                    </div>
                    <div class="footer__content__link__logo">
                        <img src="./assets/footer-logos/wingate.png" alt="wingate logo">
                    </div>
                </div>
            </div>
        </div>
        <div class="background">
            <div class="background_video">
                <video ref="backgroundVideoRef" autoplay muted loop class="blurred-video" src="/assets/background-videos/intro.mp4">
                    Your browser does not support the video tag.
                </video>
                <video ref="clearVideoRef" autoplay muted loop class="clear-video" :style="maskStyle" src="/assets/background-videos/intro.mp4">
                    Your browser does not support the video tag.
                </video>
                <div class="video-overlay"></div>
            </div>
        </div>
    </div>
</template>

<style scoped lang="scss">
@use './assets/_variables.scss' as *;

@font-face {
    font-family: $font-family;
    src: url($path-editor-sans-pro-light) format('truetype');
    font-weight: 200;
    font-style: normal;
}

@font-face {
    font-family: $font-family;
    src: url($path-editor-sans-pro-light-italic) format('truetype');
    font-weight: 200;
    font-style: italic;
}

@font-face {
    font-family: $font-family;
    src: url($path-editor-sans-pro-book) format('truetype');
    font-weight: 350;
    font-style: normal;
}

@font-face {
    font-family: $font-family;
    src: url($path-editor-sans-pro-medium) format('truetype');
    font-weight: 500;
    font-style: normal;
}

@font-face {
    font-family: $font-family;
    src: url($path-editor-sans-pro-bold-italic) format('truetype');
    font-weight: bold;
    font-style: italic;
}

@font-face {
    font-family: $font-family;
    src: url($path-editor-sans-pro-black) format('truetype');
    font-weight: 900;
    font-style: normal;
}

@font-face {
    font-family: $font-family;
    src: url($path-editor-sans-pro-ultra) format('truetype');
    font-weight: 1000;
    font-style: normal;
}

* {
    font-family: $font-family;
}

html, body, #app {
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

.navbar {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    z-index: 10; // higher than .background_video
    background: transparent; // or set a color if you want
    padding: 30px 60px;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    box-sizing: border-box;

    &__ariela {
        transition: color 0.5s cubic-bezier(0.4, 0, 0.2, 1);
        width: 97px; // Adjust as needed
        height: 59px; // Adjust as needed
        display: flex;
        align-items: center;
        justify-content: center;
        fill: white;
        color: white;

        svg {
            width: 100%;
            height: 100%;
            fill: white;
            transition: fill 0.5s cubic-bezier(0.4, 0, 0.2, 1);
        }

        &--active {
            svg {
                fill: $color-primary !important;
            }
        }
    }

    &__logo {
        position: fixed;
        left: 50vw;
        top: 50vh;
        transform: translate(-50%, -50%) scale(2.25);
        z-index: 20;
        font-weight: bold;
        font-style: italic;
        display: flex;
        flex-direction: column;
        align-items: center;
        color: white;
        text-align: center;
        font-family: $font-family;
        gap: 3px;
        transition: top 0.5s cubic-bezier(0.4, 0, 0.2, 1),
        transform 0.5s cubic-bezier(0.4, 0, 0.2, 1);

        &--active {
            color: $color-primary !important;

            svg {
                fill: $color-primary !important;
            }
        }

        &__top {
            display: flex;
            flex-direction: column;
            align-items: center;
            font-size: $text-size-md;
            line-height: $text-size-sm;
        }

        &__bottom {
            display: flex;
            flex-direction: row-reverse;
            align-items: center;
            font-size: $text-size-xs;
            gap: 6px;

            &__arrow {
                svg {
                    fill: white;
                    color: white;
                }
            }
        }
    }

    &__buy_tickets {
        display: flex;
        align-items: center;
        justify-content: center;
        font-family: $font-family;
        font-weight: bold;
        font-style: italic;
        font-size: $text-size-md;
        cursor: pointer;
        border-radius: 50%;
        color: white;
        margin-top: -10px;

        &--active {
            color: $color-primary !important;
        }
    }

    .navbar__ariela--active,
    .navbar__buy_tickets--active {
        color: $color-primary !important;
    }

    .navbar__ariela--active svg,
    .navbar__buy_tickets--active svg {
        fill: $color-primary !important;
    }
}

.content-wrapper {
    position: relative;
    width: 100%;
    min-height: 450vh;
    top: 100vh;
}

.content-spacer {
    height: 10vh;
    width: 100%;
}

.content {
    position: sticky;
    top: 240px;
    display: flex;
    flex-direction: row;
    justify-content: center;
    width: 100%;
    z-index: 1;
    padding: 0 60px;
    box-sizing: border-box;
    font-family: $font-family;
    direction: rtl;
    align-items: flex-start;
    max-height: calc(100vh - 240px);

    &__menu {
        display: flex;
        flex-direction: column;
        gap: 25px;
        width: 200px;
        color: white;
        position: relative;
        z-index: 2;

        &__item {
            cursor: pointer;
            transition: color 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            font-size: $text-size-md;
            font-weight: 500;
            line-height: 38px;
            position: relative;
            z-index: 1;
            will-change: transform, opacity;
            transform: translateZ(0);
            backface-visibility: hidden;
            -webkit-font-smoothing: antialiased;

            &:hover {
                color: $color-primary;
                transition: color 0.2s cubic-bezier(0.4, 0, 0.2, 1);
            }

            &--active {
                text-decoration: underline;
            }
        }
    }

    &__main {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: flex-start;
        width: 100%;
        gap: 50px;
        position: relative;
        z-index: 1;

        &__item {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            width: 100%;
            gap: 20px;
            max-width: 1200px;
            box-sizing: border-box;
            position: absolute;
            top: 0;
            left: 50%;
            transform: translateX(-50%) translateZ(0);
            will-change: opacity, transform;
            backface-visibility: hidden;
            -webkit-font-smoothing: antialiased;
            z-index: 1;
            pointer-events: none;
            cursor: default;

            &[style*="opacity: 1"] {
                z-index: 2;
                pointer-events: auto;

                .content__main__item__slot {
                    pointer-events: auto;


                    &__wrapper {
                        pointer-events: auto;
                    }

                    &__description {
                        pointer-events: auto;
                        cursor: default;
                        display: flex;
                        flex-direction: column;

                        &__button {
                            pointer-events: auto;
                            cursor: pointer;
                            color: white;
                            font-weight: 300;
                            font-size: $text-size-xs;
                            line-height: 25px;
                            display: flex;
                            flex-direction: row;
                            justify-content: flex-start;
                            gap: 4px;
                            font-style: italic;
                            align-items: center;

                            &:hover {
                                color: $color-primary;
                                transition: color 0.2s cubic-bezier(0.4, 0, 0.2, 1);

                                svg, path, rect {
                                    fill: $color-primary;
                                    transition: fill 0.2s cubic-bezier(0.4, 0, 0.2, 1);
                                }
                            }
                        }
                    }
                }
            }

            &__description, &__slot {
                pointer-events: auto;
            }

            &__description {
                font-size: $text-size-md;
                color: white;
                font-weight: 300;
                line-height: 40px;
                width: 732px;
                will-change: transform, opacity, filter;
                transform-origin: center center;
                backface-visibility: hidden;
                -webkit-font-smoothing: antialiased;
                transform: translateZ(0);
                margin-top: 1px;
                padding-right: -90px;
                cursor: default;

                .highlight-content-text {
                    color: white;
                    cursor: pointer;
                    transition: color 0.2s cubic-bezier(0.4, 0, 0.2, 1);

                    &:hover {
                        color: $color-primary;
                        transition: color 0.2s cubic-bezier(0.4, 0, 0.2, 1);
                    }
                }
            }

            &__slot {
                display: flex;
                flex-direction: column;
                justify-content: flex-start;
                color: white;
                width: 700px;
                position: relative;
                transform: translateZ(0);
                will-change: transform, opacity, filter, scale;
                backface-visibility: hidden;
                -webkit-font-smoothing: antialiased;
                transition: color 0.3s cubic-bezier(0.4, 0, 0.2, 1);
                pointer-events: none;

                &:hover {
                    cursor: pointer;
                    color: $color-primary;
                    transition: color 0.2s cubic-bezier(0.4, 0, 0.2, 1);
                }

                &__wrapper {
                    display: flex;
                    flex-direction: row;
                    align-items: flex-end;
                    justify-content: flex-start;
                    gap: 70px;
                    transform: translateZ(0);
                    will-change: transform;
                    pointer-events: none;
                }

                &__description {
                    display: flex;
                    padding-right: 123px;
                    top: 100%;
                    color: white;
                    font-size: $text-size-xs;
                    font-weight: 300;
                    line-height: 18px;
                    transform: translateZ(0);
                    will-change: transform, opacity;
                    pointer-events: none;
                    width: 347px;
                    margin-bottom: 5px;
                }

                &__time {
                    font-size: $text-size-xs;
                    font-weight: 300;
                    line-height: 38px;
                    width: 52px;
                    display: flex;
                    flex-direction: row;
                    gap: 1px;

                }

                &__info {
                    display: flex;
                    flex-direction: column;
                    gap: 0;

                    &__speakers {
                        font-size: $text-size-xs;
                        font-weight: 300;
                        margin-bottom: -1px;
                    }

                    &__title {
                        font-size: $text-size-md;
                        font-weight: 500;
                        line-height: 38px;
                    }
                }
            }
        }
    }

    &__spacer {
        height: 100vh;
    }

    &__item {
        display: flex;
        flex-direction: row-reverse;
        align-items: flex-start;
        justify-content: flex-start;
        gap: 210px;
        width: 100%;
        padding: 50px 60px;
        box-sizing: border-box;

        &__title {
            font-size: $text-size-md;
            font-weight: bold;
            font-style: italic;
            color: white;
        }

        &__descrtiption {
            font-size: $text-size-md;
            color: white;
            font-weight: bold;
            font-style: italic;
            line-height: 38px;
            width: 700px;
            direction: rtl;
        }

        &__schedule {
            display: flex;
            flex-direction: column;
            gap: 28px;
            color: white;
            direction: rtl;

            &__slot {
                display: flex;
                flex-direction: column;
                color: white;
                transition: color 0.3s cubic-bezier(0.4, 0, 0.2, 1);
                width: 470px;
                justify-content: flex-start;

                &:hover {
                    cursor: pointer;
                    color: $color-primary;
                    transition: color 0.2s cubic-bezier(0.4, 0, 0.2, 1);
                }

                &__wrapper {
                    display: flex;
                    flex-direction: row;
                    align-items: flex-end;
                    gap: 80px;

                }


                &__time {
                    font-size: $text-size-xs;
                    font-weight: 500;
                    line-height: 38px;
                }

                &__main {
                    display: flex;
                    flex-direction: column;
                    gap: 0;

                    &__speakers {
                        font-size: $text-size-xs;
                        font-weight: 500;
                        line-height: 25px;
                    }

                    &__title {
                        font-size: $text-size-md;
                        font-weight: 500;
                        line-height: 25px;
                    }

                    &__description {
                        width: 100%;
                        padding-top: 10px;
                        font-size: $text-size-xs;
                        font-weight: 500;
                        color: white;
                        line-height: 18px;
                    }
                }
            }
        }
    }
}

.background {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: -1;
    overflow: hidden;
    background: #000;
}

.background_video {
    position: relative;
    width: 100vw;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
}

.blurred-video, .clear-video {
    position: absolute;
    width: 1511px;
    height: 873px;
    left: -31px;
    top: 2px;
    object-fit: cover;
    display: block;
}

.clear-video {
    /* This video will be revealed by the mask */
    z-index: 1;
    will-change: mask;
    transition: mask 0.8s cubic-bezier(0.16, 1, 0.3, 1), -webkit-mask 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}

.blurred-video {
    /* Your GSAP blur will be applied to this video */
    z-index: 0;
}

.boycott_modal {
    position: fixed;
    direction: rtl;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    color: white;
    background: rgba(0, 0, 0, 0.8);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9;
    overflow: hidden;

    &__content {
        display: flex;
        flex-direction: row;
        width: 1100px;
        height: 408px;
        gap: 10px;

        &__right {
            width: 50%;
            position: relative;
            //overflow: hidden;

            &__image {
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                object-fit: cover;
                opacity: 1;
            }

            &__back {
                position: absolute;
                top: -40px;
                right: 0;
                cursor: pointer;
                transition: all 0.3s ease;
                font-family: $font-family;
                display: flex;
                justify-content: flex-start;
                z-index: 2;
                width: 100%;
            }
        }

        &__left {
            width: 50%;
            padding: 30px 30px 30px 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: flex-end;
            gap: 20px;

            &__title {
                font-size: $text-size-md;
                font-weight: bold;
                line-height: 36px;
                color: $color-primary;
                font-style: italic;
            }

            &__text {
                font-size: $text-size-sm;
                line-height: 32px;
                color: white;
                font-weight: 300;
            }
        }
    }
}

.spinning-circle {
    width: 70px;
    height: 70px;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    margin-bottom: -6px;

    &__svg {
        position: relative;
        z-index: 1;
        width: 100%;
        height: 100%;
        animation: spin 10s linear infinite;
        fill: white;
    }

    &:hover {
        transform: scale(1.2);
        transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);

        text {
            fill: $color-primary;
        }

    }

    text {
        font-size: 24px;
        letter-spacing: 1px;
        text-transform: uppercase;
        fill: currentColor;
        font-family: $font-family;
    }
}

@keyframes spin {
    100% {
        transform: rotate(-360deg);
    }
}

.spacer {
    height: 200vh;
}

.highlight-content-text {
    color: white;
    font-weight: bold;
    font-style: italic;

    &:hover {
        cursor: pointer;
        color: $color-primary;
        transition: color 0.2s cubic-bezier(0.4, 0, 0.2, 1);
    }

}

.active {
    color: $color-primary !important;
}

.exclusion-element {
    //cursor: pointer;
}

.first_day {
    margin-top: 70px;
    padding-right: 128px; // TODO: make it native
}

.second_day {
    margin-top: 111px;
    padding-right: 128px;
}

.video-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: black;
    opacity: 0;
    z-index: 1;
    pointer-events: none;
    transition: opacity 0.3s ease;
}

.book_pages {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.9);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9;
    overflow: hidden;

    &__content {
        position: relative;
        display: flex;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        margin-top: 60px;
        gap: 20px;

        &__image {
            max-width: 90%;
            max-height: 90%;
            width: 900px;
            object-fit: contain;
            cursor: pointer;
            transition: transform 0.3s ease;

            &:hover {
                transform: scale(1.02);
            }
        }
    }
}

.footer {
    position: fixed;
    bottom: 0;
    left: 0;
    padding: 30px 60px;
    width: 100%;
    color: white;
    z-index: 10;
    opacity: 0;
    transform: translateY(100px);
    will-change: transform, opacity;
    backface-visibility: hidden;
    -webkit-font-smoothing: antialiased;
    box-sizing: border-box;

    &__content {
        display: flex;
        justify-content: space-between;
        align-items: flex-end;
        flex-direction: column;
        gap: 10px;
        font-weight: 350;

        &__line {
            font-size: $text-size-xs;
            line-height: 20px;
            color: white;
            display: flex;
            flex-direction: row-reverse;
            gap: 24px;

            &__logo {
                height: 23px;
            }
        }

    }
}
</style>