---
layout: default
---

<div class="page-header position-relative">
    <div class="container">
        <h1 class="page-heading single-col-max mx-auto">R<span id="feature-text" class="text-alt"></span>ish</h1>
        <div class="page-intro single-col-max mx-auto">Declarative, element-based UI library for Unity.</div>
        <div class="single-col-max mx-auto mt-5">
            <a href="{{ "/docs/rish/" | append: site.rish-versions[0] | append: "/quick-start" }}" class="btn btn-light btn-sm">Learn Rish</a>
        </div>
    </div>
</div><!--//page-header-->
<div class="page-content">
    <div class="docs-overview container py-5 justify-content-center">
        {% assign pages = site.pages | sort: "order" %}
        {% for page in pages %}
            {% if page.feature %}
            <div class="feature-highlight d-flex flex-column">
                <h2>{{ page.title }}</h2>
                {{ page.content }}
            </div>
            {% endif %}
        {% endfor %}
    </div>
</div><!--//page-content-->

<section class="cta-section text-center py-5 theme-bg-dark position-relative">
    <div class="container d-flex flex-row justify-content-center align-items-center">
        <div class="d-flex flex-column align-items-center">
            <h2>
                <span class="small">Best of</span>
                Retained Mode
            </h2>
            <i class="fa-solid fa-plus"></i>
            <h2>
                <span class="small">Best of</span>
                Immediate Mode
            </h2>
        </div>
        <i class="fa-solid fa-equals m-3"></i>
        <h1>Rish</h1>
    </div>
</section><!--//cta-section-->

{% include footer.html %}

<!-- Javascript -->
<script src="/assets/plugins/popper.min.js"></script>
<script src="/assets/plugins/bootstrap/js/bootstrap.min.js"></script>

<!-- Page Specific JS -->
<script src="/assets/plugins/smoothscroll.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.15.8/highlight.min.js"></script>
<script src="/assets/js/highlight-custom.js"></script>
<script src="/assets/plugins/simplelightbox/simple-lightbox.min.js"></script>
<script src="/assets/plugins/gumshoe/gumshoe.polyfills.min.js"></script>
<script src="/assets/js/docs.js"></script>

<script defer>
    type("feature-text");

    async function typeSentence(sentence, element, delay = 100) {
        const letters = sentence.split("");
        let i = 0;
        while (i < letters.length) {
            await waitForMs(delay);

            element.innerHTML += letters[i];

            i++;
        }
        return;
    }

    async function deleteSentence(element) {
        const sentence = element.innerHTML;
        const letters = sentence.split("");
        let i = 0;
        while (letters.length > 0) {
            await waitForMs(100);
            letters.pop();

            element.innerHTML = letters.join("");
        }
    }

    async function type(elementId) {
        var element = document.getElementById(elementId);

        while (true) {
            await waitForMs(5000);
            await typeSentence("eact", element);
            await waitForMs(1500);
            await deleteSentence(element);
        }
    }

    function waitForMs(ms) {
        return new Promise(resolve => setTimeout(resolve, ms))
    }
</script>