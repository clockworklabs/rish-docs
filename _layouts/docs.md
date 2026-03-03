---
layout: default
---
{% assign url = page.url | split: "/" %}
{% assign library = url[2] %}
{% assign version = url[3] %}
{% if version contains "." %}
	{% assign has_version = true %}
{% else %}
	{% assign has_version = false %}
{% endif %}
{% assign other_library = "" %}
{% assign other_version = "" %}
{% if library == "rish" %}
	{% assign other_library = "roots" %}
	{% if has_version %}
		{% for pair in site.rish-roots-pairs %}
			{% assign rish_version = pair[0] %}
			{% if rish_version == version %}
				{% assign other_version = pair[1] %}
				{% break %}
			{% endif %}
		{% endfor %}
	{% else %}
		{% assign version = site.rish-versions[0] %}
	{% endif %}
{% else %}
	{% assign other_library = "rish" %}
	{% if has_version %}
		{% for pair in site.rish-roots-pairs %}
			{% assign roots_version = pair[1] %}
			{% if roots_version == version %}
				{% assign other_version = pair[0] %}
				{% break %}
			{% endif %}
		{% endfor %}
	{% else %}
		{% assign version = site.roots-versions[0] %}
	{% endif %}
{% endif %}
{% capture target_url %}/docs/{{ library }}{% if has_version %}/{{ version }}{% endif %}{% endcapture %}
{% capture other_target_url %}/docs/{{ other_library }}{% if has_version %}/{{ other_version }}{% endif %}{% endcapture %}
{% capture library_version %}{{ library }}/{{ version }}{% endcapture %}
{% capture full_url %}{% if has_version %}{{ page.url }}{% else %}{{ page.url | replace: library, library_version }}{% endif %}{% endcapture %}
<div class="docs-wrapper">
	<div id="docs-sidebar" class="docs-sidebar">
		<nav id="docs-nav" class="docs-nav navbar">
			<ul class="section-items list-unstyled nav flex-column pb-3">
				{% assign i = 0 %}
				{% assign pages = site.docs | where_exp: "doc", "doc.categories contains library" | where_exp: "doc", "doc.categories contains version" %}
			  	{% for doc in pages %}
					<li class="nav-item section-title{% if doc.slug == page.slug %} active{% endif %} {% if i > 0 %} mt-3{% endif %}"><a class="nav-link" href="{{ target_url }}/{{ doc.slug }}"><span class="theme-icon-holder me-2"><i class="fa-{{ doc.icon-style }} fa-{% if doc.icon %}{{ doc.icon }}{% else %}right-long{% endif %}"></i></span>{{ doc.title }}</a></li>
					{% if doc.slug == page.slug %}
						{% for doc-section in doc.sections %}
							<li class="nav-item"><a class="nav-link scrollto" href="#{{ doc-section | slugify }}">{{ doc-section }}</a></li>
						{% endfor %}
					{% endif %}
					{% assign i = i | plus:1 %}
  				{% endfor %}
			</ul>
		</nav><!--//docs-nav-->
		<div class="dropdown version-picker">
			<button class="btn btn-secondary btn-sm dropdown-toggle" type="button" data-bs-toggle="dropdown" data-bs-offset="0,0" aria-expanded="false">
				{{ version }}
			</button>
			<ul class="dropdown-menu">
				{% if library == "rish" %}
					{% for v in site.rish-versions %}
						<li><a class="dropdown-item" href="{{ full_url | replace: version, v }}">{{ v }}</a></li>
					{% endfor %}
				{% else %}
					{% for v in site.roots-versions %}
						<li><a class="dropdown-item" href="{{ full_url | replace: version, v }}">{{ v }}</a></li>
					{% endfor %}
				{% endif %}
			</ul>
		</div>
		<a href="{{ other_target_url | append: "/quick-start" }}" class="btn btn-light btn-sm m-3">{{ other_library | capitalize }} Docs</a>
	</div><!--//docs-sidebar-->
	<div class="docs-content">
		<div class="container">
			<article class="docs-article" id="section-1">
				<header class="docs-header">
					<h1 id={{ page.title | slugify }} class="docs-heading">{{ page.title }}</h1>
				</header>
				{{ content }}
			</article>
			{% include footer.html %}
		</div> 
	</div>
</div><!--//docs-wrapper-->

	
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
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

<script defer>
	// Get the header
	var header = document.querySelector('#navbar-header');

	// Initialize Gumshoe
	var spy = new Gumshoe('#docs-sidebar a', {
		offset: function () {
			return header.getBoundingClientRect().height;
		}
	});
</script>