<!doctype html>
<html class="no-js" lang="{{ request.locale.iso_code }}">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <meta name="theme-color" content="">
    <link rel="canonical" href="{{ canonical_url }}">

    {%- if settings.favicon != blank -%}
      <link rel="icon" type="image/png" href="{{ settings.favicon | image_url: width: 32, height: 32 }}">
    {%- endif -%}

    {%- unless settings.type_header_font.system? and settings.type_body_font.system? -%}
      <link rel="preconnect" href="https://fonts.shopifycdn.com" crossorigin>
    {%- endunless -%}

    <title>
      {{ page_title }}
      {%- if current_tags %} &ndash; tagged "{{ current_tags | join: ', ' }}"{% endif -%}
      {%- if current_page != 1 %} &ndash; Page {{ current_page }}{% endif -%}
      {%- unless page_title contains shop.name %} &ndash; {{ shop.name }}{% endunless -%}
    </title>

    {% if page_description %}
      <meta name="description" content="{{ page_description | escape }}">
    {% endif %}

    {% render 'meta-tags' %}
    <script src="https://code.jquery.com/jquery-3.6.4.min.js"></script>
    <script src="{{ 'constants.js' | asset_url }}" defer="defer"></script>
    <script src="{{ 'pubsub.js' | asset_url }}" defer="defer"></script>
    <script src="{{ 'global.js' | asset_url }}" defer="defer"></script>
    {%- if settings.animations_reveal_on_scroll -%}
      <script src="{{ 'animations.js' | asset_url }}" defer="defer"></script>
    {%- endif -%}

    {{ content_for_header }}

    {%- liquid
      assign body_font_bold = settings.type_body_font | font_modify: 'weight', 'bold'
      assign body_font_italic = settings.type_body_font | font_modify: 'style', 'italic'
      assign body_font_bold_italic = body_font_bold | font_modify: 'style', 'italic'
    %}

    {% style %}
      {{ settings.type_body_font | font_face: font_display: 'swap' }}
      {{ body_font_bold | font_face: font_display: 'swap' }}
      {{ body_font_italic | font_face: font_display: 'swap' }}
      {{ body_font_bold_italic | font_face: font_display: 'swap' }}
      {{ settings.type_header_font | font_face: font_display: 'swap' }}

      {% for scheme in settings.color_schemes -%}
        {% assign scheme_classes = scheme_classes | append: ', .color-' | append: scheme.id %}
        {% if forloop.index == 1 -%}
          :root,
        {%- endif %}
        .color-{{ scheme.id }} {
          --color-background: {{ scheme.settings.background.red }},{{ scheme.settings.background.green }},{{ scheme.settings.background.blue }};
        {% if scheme.settings.background_gradient != empty %}
          --gradient-background: {{ scheme.settings.background_gradient }};
        {% else %}
          --gradient-background: {{ scheme.settings.background }};
        {% endif %}

        {% liquid
          assign background_color = scheme.settings.background
          assign background_color_brightness = background_color | color_brightness
          if background_color_brightness <= 26
            assign background_color_contrast = background_color | color_lighten: 50
          elsif background_color_brightness <= 65
            assign background_color_contrast = background_color | color_lighten: 5
          else
            assign background_color_contrast = background_color | color_darken: 25
          endif
        %}

        --color-foreground: {{ scheme.settings.text.red }},{{ scheme.settings.text.green }},{{ scheme.settings.text.blue }};
        --color-background-contrast: {{ background_color_contrast.red }},{{ background_color_contrast.green }},{{ background_color_contrast.blue }};
        --color-shadow: {{ scheme.settings.shadow.red }},{{ scheme.settings.shadow.green }},{{ scheme.settings.shadow.blue }};
        --color-button: {{ scheme.settings.button.red }},{{ scheme.settings.button.green }},{{ scheme.settings.button.blue }};
        --color-button-text: {{ scheme.settings.button_label.red }},{{ scheme.settings.button_label.green }},{{ scheme.settings.button_label.blue }};
        --color-secondary-button: {{ scheme.settings.background.red }},{{ scheme.settings.background.green }},{{ scheme.settings.background.blue }};
        --color-secondary-button-text: {{ scheme.settings.secondary_button_label.red }},{{ scheme.settings.secondary_button_label.green }},{{ scheme.settings.secondary_button_label.blue }};
        --color-link: {{ scheme.settings.secondary_button_label.red }},{{ scheme.settings.secondary_button_label.green }},{{ scheme.settings.secondary_button_label.blue }};
        --color-badge-foreground: {{ scheme.settings.text.red }},{{ scheme.settings.text.green }},{{ scheme.settings.text.blue }};
        --color-badge-background: {{ scheme.settings.background.red }},{{ scheme.settings.background.green }},{{ scheme.settings.background.blue }};
        --color-badge-border: {{ scheme.settings.text.red }},{{ scheme.settings.text.green }},{{ scheme.settings.text.blue }};
        --payment-terms-background-color: rgb({{ scheme.settings.background.rgb }});
      }
      {% endfor %}


      {{ scheme_classes | prepend: 'body' }} {
        color: rgba(var(--color-foreground), 0.75);
        background-color: rgb(var(--color-background));
      }


      :root {
        --font-body-family: {{ settings.type_body_font.family }}, {{ settings.type_body_font.fallback_families }};
        --font-body-style: {{ settings.type_body_font.style }};
        --font-body-weight: {{ settings.type_body_font.weight }};
        --font-body-weight-bold: {{ settings.type_body_font.weight | plus: 300 | at_most: 1000 }};

        --font-heading-family: {{ settings.type_header_font.family }}, {{ settings.type_header_font.fallback_families }};
        --font-heading-style: {{ settings.type_header_font.style }};
        --font-heading-weight: {{ settings.type_header_font.weight }};

        --font-body-scale: {{ settings.body_scale | divided_by: 100.0 }};
        --font-heading-scale: {{ settings.heading_scale | times: 1.0 | divided_by: settings.body_scale }};

        --media-padding: {{ settings.media_padding }}px;
        --media-border-opacity: {{ settings.media_border_opacity | divided_by: 100.0 }};
        --media-border-width: {{ settings.media_border_thickness }}px;
        --media-radius: {{ settings.media_radius }}px;
        --media-shadow-opacity: {{ settings.media_shadow_opacity | divided_by: 100.0 }};
        --media-shadow-horizontal-offset: {{ settings.media_shadow_horizontal_offset }}px;
        --media-shadow-vertical-offset: {{ settings.media_shadow_vertical_offset }}px;
        --media-shadow-blur-radius: {{ settings.media_shadow_blur }}px;
        --media-shadow-visible: {% if settings.media_shadow_opacity > 0 %}1{% else %}0{% endif %};

        --page-width: {{ settings.page_width | divided_by: 10 }}rem;
        --page-width-margin: {% if settings.page_width == '1600' %}2{% else %}0{% endif %}rem;

        --product-card-image-padding: {{ settings.card_image_padding | divided_by: 10.0 }}rem;
        --product-card-corner-radius: {{ settings.card_corner_radius | divided_by: 10.0 }}rem;
        --product-card-text-alignment: {{ settings.card_text_alignment }};
        --product-card-border-width: {{ settings.card_border_thickness | divided_by: 10.0 }}rem;
        --product-card-border-opacity: {{ settings.card_border_opacity | divided_by: 100.0 }};
        --product-card-shadow-opacity: {{ settings.card_shadow_opacity | divided_by: 100.0 }};
        --product-card-shadow-visible: {% if settings.card_shadow_opacity > 0 %}1{% else %}0{% endif %};
        --product-card-shadow-horizontal-offset: {{ settings.card_shadow_horizontal_offset | divided_by: 10.0 }}rem;
        --product-card-shadow-vertical-offset: {{ settings.card_shadow_vertical_offset | divided_by: 10.0 }}rem;
        --product-card-shadow-blur-radius: {{ settings.card_shadow_blur | divided_by: 10.0 }}rem;

        --collection-card-image-padding: {{ settings.collection_card_image_padding | divided_by: 10.0 }}rem;
        --collection-card-corner-radius: {{ settings.collection_card_corner_radius | divided_by: 10.0 }}rem;
        --collection-card-text-alignment: {{ settings.collection_card_text_alignment }};
        --collection-card-border-width: {{ settings.collection_card_border_thickness | divided_by: 10.0 }}rem;
        --collection-card-border-opacity: {{ settings.collection_card_border_opacity | divided_by: 100.0 }};
        --collection-card-shadow-opacity: {{ settings.collection_card_shadow_opacity | divided_by: 100.0 }};
        --collection-card-shadow-visible: {% if settings.collection_card_shadow_opacity > 0 %}1{% else %}0{% endif %};
        --collection-card-shadow-horizontal-offset: {{ settings.collection_card_shadow_horizontal_offset | divided_by: 10.0 }}rem;
        --collection-card-shadow-vertical-offset: {{ settings.collection_card_shadow_vertical_offset | divided_by: 10.0 }}rem;
        --collection-card-shadow-blur-radius: {{ settings.collection_card_shadow_blur | divided_by: 10.0 }}rem;

        --blog-card-image-padding: {{ settings.blog_card_image_padding | divided_by: 10.0 }}rem;
        --blog-card-corner-radius: {{ settings.blog_card_corner_radius | divided_by: 10.0 }}rem;
        --blog-card-text-alignment: {{ settings.blog_card_text_alignment }};
        --blog-card-border-width: {{ settings.blog_card_border_thickness | divided_by: 10.0 }}rem;
        --blog-card-border-opacity: {{ settings.blog_card_border_opacity | divided_by: 100.0 }};
        --blog-card-shadow-opacity: {{ settings.blog_card_shadow_opacity | divided_by: 100.0 }};
        --blog-card-shadow-visible: {% if settings.blog_card_shadow_opacity > 0 %}1{% else %}0{% endif %};
        --blog-card-shadow-horizontal-offset: {{ settings.blog_card_shadow_horizontal_offset | divided_by: 10.0 }}rem;
        --blog-card-shadow-vertical-offset: {{ settings.blog_card_shadow_vertical_offset | divided_by: 10.0 }}rem;
        --blog-card-shadow-blur-radius: {{ settings.blog_card_shadow_blur | divided_by: 10.0 }}rem;

        --badge-corner-radius: {{ settings.badge_corner_radius | divided_by: 10.0 }}rem;

        --popup-border-width: {{ settings.popup_border_thickness }}px;
        --popup-border-opacity: {{ settings.popup_border_opacity | divided_by: 100.0 }};
        --popup-corner-radius: {{ settings.popup_corner_radius }}px;
        --popup-shadow-opacity: {{ settings.popup_shadow_opacity | divided_by: 100.0 }};
        --popup-shadow-horizontal-offset: {{ settings.popup_shadow_horizontal_offset }}px;
        --popup-shadow-vertical-offset: {{ settings.popup_shadow_vertical_offset }}px;
        --popup-shadow-blur-radius: {{ settings.popup_shadow_blur }}px;

        --drawer-border-width: {{ settings.drawer_border_thickness }}px;
        --drawer-border-opacity: {{ settings.drawer_border_opacity | divided_by: 100.0 }};
        --drawer-shadow-opacity: {{ settings.drawer_shadow_opacity | divided_by: 100.0 }};
        --drawer-shadow-horizontal-offset: {{ settings.drawer_shadow_horizontal_offset }}px;
        --drawer-shadow-vertical-offset: {{ settings.drawer_shadow_vertical_offset }}px;
        --drawer-shadow-blur-radius: {{ settings.drawer_shadow_blur }}px;

        --spacing-sections-desktop: {{ settings.spacing_sections }}px;
        --spacing-sections-mobile: {% if settings.spacing_sections < 24 %}{{ settings.spacing_sections }}{% else %}{{ settings.spacing_sections | times: 0.7 | round | at_least: 20 }}{% endif %}px;

        --grid-desktop-vertical-spacing: {{ settings.spacing_grid_vertical }}px;
        --grid-desktop-horizontal-spacing: {{ settings.spacing_grid_horizontal }}px;
        --grid-mobile-vertical-spacing: {{ settings.spacing_grid_vertical | divided_by: 2 }}px;
        --grid-mobile-horizontal-spacing: {{ settings.spacing_grid_horizontal | divided_by: 2 }}px;

        --text-boxes-border-opacity: {{ settings.text_boxes_border_opacity | divided_by: 100.0 }};
        --text-boxes-border-width: {{ settings.text_boxes_border_thickness }}px;
        --text-boxes-radius: {{ settings.text_boxes_radius }}px;
        --text-boxes-shadow-opacity: {{ settings.text_boxes_shadow_opacity | divided_by: 100.0 }};
        --text-boxes-shadow-visible: {% if settings.text_boxes_shadow_opacity > 0 %}1{% else %}0{% endif %};
        --text-boxes-shadow-horizontal-offset: {{ settings.text_boxes_shadow_horizontal_offset }}px;
        --text-boxes-shadow-vertical-offset: {{ settings.text_boxes_shadow_vertical_offset }}px;
        --text-boxes-shadow-blur-radius: {{ settings.text_boxes_shadow_blur }}px;

        --buttons-radius: {{ settings.buttons_radius }}px;
        --buttons-radius-outset: {% if settings.buttons_radius > 0 %}{{ settings.buttons_radius | plus: settings.buttons_border_thickness }}{% else %}0{% endif %}px;
        --buttons-border-width: {% if settings.buttons_border_opacity > 0 %}{{ settings.buttons_border_thickness }}{% else %}0{% endif %}px;
        --buttons-border-opacity: {{ settings.buttons_border_opacity | divided_by: 100.0 }};
        --buttons-shadow-opacity: {{ settings.buttons_shadow_opacity | divided_by: 100.0 }};
        --buttons-shadow-visible: {% if settings.buttons_shadow_opacity > 0 %}1{% else %}0{% endif %};
        --buttons-shadow-horizontal-offset: {{ settings.buttons_shadow_horizontal_offset }}px;
        --buttons-shadow-vertical-offset: {{ settings.buttons_shadow_vertical_offset }}px;
        --buttons-shadow-blur-radius: {{ settings.buttons_shadow_blur }}px;
        --buttons-border-offset: {% if settings.buttons_radius > 0 or settings.buttons_shadow_opacity > 0 %}0.3{% else %}0{% endif %}px;

        --inputs-radius: {{ settings.inputs_radius }}px;
        --inputs-border-width: {{ settings.inputs_border_thickness }}px;
        --inputs-border-opacity: {{ settings.inputs_border_opacity | divided_by: 100.0 }};
        --inputs-shadow-opacity: {{ settings.inputs_shadow_opacity | divided_by: 100.0 }};
        --inputs-shadow-horizontal-offset: {{ settings.inputs_shadow_horizontal_offset }}px;
        --inputs-margin-offset: {% if settings.inputs_shadow_vertical_offset != 0 and settings.inputs_shadow_opacity > 0 %}{{ settings.inputs_shadow_vertical_offset | abs }}{% else %}0{% endif %}px;
        --inputs-shadow-vertical-offset: {{ settings.inputs_shadow_vertical_offset }}px;
        --inputs-shadow-blur-radius: {{ settings.inputs_shadow_blur }}px;
        --inputs-radius-outset: {% if settings.inputs_radius > 0 %}{{ settings.inputs_radius | plus: settings.inputs_border_thickness }}{% else %}0{% endif %}px;

        --variant-pills-radius: {{ settings.variant_pills_radius }}px;
        --variant-pills-border-width: {{ settings.variant_pills_border_thickness }}px;
        --variant-pills-border-opacity: {{ settings.variant_pills_border_opacity | divided_by: 100.0 }};
        --variant-pills-shadow-opacity: {{ settings.variant_pills_shadow_opacity | divided_by: 100.0 }};
        --variant-pills-shadow-horizontal-offset: {{ settings.variant_pills_shadow_horizontal_offset }}px;
        --variant-pills-shadow-vertical-offset: {{ settings.variant_pills_shadow_vertical_offset }}px;
        --variant-pills-shadow-blur-radius: {{ settings.variant_pills_shadow_blur }}px;
      }

      *,
      *::before,
      *::after {
        box-sizing: inherit;
      }


      html {
        box-sizing: border-box;
        font-size: calc(var(--font-body-scale) * 62.5%);
        height: 100%;
      }

      body {
        display: grid;
        grid-template-rows: auto auto 1fr auto;
        grid-template-columns: 100%;
        min-height: 100%;
        margin: 0;
        font-size: 1.5rem;
        letter-spacing: 0.06rem;
        line-height: calc(1 + 0.8 / var(--font-body-scale));
        font-family: var(--font-body-family);
        font-style: var(--font-body-style);
        font-weight: var(--font-body-weight);
      }

      @media screen and (min-width: 750px) {
        body {
          font-size: 1.6rem;
        }
      }
    {% endstyle %}

    {{ 'base.css' | asset_url | stylesheet_tag }}

    {%- unless settings.type_body_font.system? -%}
      <link rel="preload" as="font" href="{{ settings.type_body_font | font_url }}" type="font/woff2" crossorigin>
    {%- endunless -%}
    {%- unless settings.type_header_font.system? -%}
      <link rel="preload" as="font" href="{{ settings.type_header_font | font_url }}" type="font/woff2" crossorigin>
    {%- endunless -%}

    {%- if localization.available_countries.size > 1 or localization.available_languages.size > 1 -%}
      {{ 'component-localization-form.css' | asset_url | stylesheet_tag: preload: true }}
      <script src="{{ 'localization-form.js' | asset_url }}" defer="defer"></script>
    {%- endif -%}

    {%- if settings.predictive_search_enabled -%}
      <link
        rel="stylesheet"
        href="{{ 'component-predictive-search.css' | asset_url }}"
        media="print"
        onload="this.media='all'"
      >
    {%- endif -%}

    <script>
      document.documentElement.className = document.documentElement.className.replace('no-js', 'js');
      if (Shopify.designMode) {
        document.documentElement.classList.add('shopify-design-mode');
      }
    </script>

    <!-- 21-04-2024 Product Page Section Css Start -->
    <style>
      .cstm-product-slider-component .cstm-prod-slide-component-ul {
        max-width: var(--page-width);
        padding: 0 5rem;
        margin-inline: auto;
      }
      .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item .multicolumn-card__image-wrapper {
          padding: 0;
          margin: 0;
          width: 40px;
          height: 40px;
          margin-right: 17px;
      }
      .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item .multicolumn-card__info {
          padding: 0;
      }
      .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item .multicolumn-card__info h3.inline-richtext {
        font-family: 'Inter', sans-serif;
        font-weight: 600;
        letter-spacing: 0;
        font-size: 16px;
        margin-bottom: 6px;
      }
      .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item .multicolumn-card__info .rte {
          margin: 0;
      }
      .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item .multicolumn-card__info p {
          letter-spacing: 0;
          font-size: 14px;
          font-family: 'Inter', sans-serif;
          font-weight: 400;
          line-height: 1.3;
      }
      .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item {
          padding-inline: 0;
      }
      .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item .multicolumn-card__info p {
          letter-spacing: 0;
          font-size: 16px;
          font-family: 'Inter', sans-serif;
          font-weight: 400;
          line-height: 1.3;
      }
      .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item {
          padding-inline: 0;
          padding-top: 0;
          margin-bottom: 0 !important;
      }
      /* Sticky Product Css */
      div#product-sticky-atc-wrapper div#product-sticky-atc {
          z-index: 100000000000000020;
          position: fixed;
          min-width: 280px;
          bottom: 15px;
          margin-bottom: 0!important;
          max-width: 350px;
          font-size: .875rem;
          background-color: rgba(255,255,255,.85);
          background-clip: padding-box;
          border: 1px solid rgba(0,0,0,.1);
          box-shadow: 0 .25rem .75rem rgba(0,0,0,.1);
          opacity: 1;
          border-radius: .25rem;
          left: 50%;
          transform: translateX(-50%);
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc .toast-header {
          display: flex;
          align-items: center;
          padding: .5rem .75rem;
          color: #6c757d;
          background-color: rgba(255,255,255,.85);
          background-clip: padding-box;
          border-bottom: 1px solid rgba(0,0,0,.05);
          border-top-left-radius: calc(.25rem - 1px);
          border-top-right-radius: calc(.25rem - 1px);
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc .toast-header .mr-auto {
          color: #6c757d;
          font-size: 14px;
          font-family: 'Inter', sans-serif;
          line-height: normal;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc .toast-header .mr-auto strong.line-height-normal {
          font-weight: 500;
          letter-spacing: 0;
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc .toast-header  button.ml-2.mb-1.close {
          padding: 0;
          background-color: transparent;
          border: 0;
          margin-left: 8px;
          font-size: 25px;
          font-weight: 500;
          line-height: 1;
          color: #000;
          text-shadow: 0 1px 0 #fff;
          opacity: .5;
          cursor: pointer;
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc  .toast-body {
        padding: 12px;
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc .toast-body img.product-img {
          width: 70px;
          margin-right: 8px;
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc .toast-body .form-group.mb-2 select.custom-select {
          display: inline-block;
          width: 100%;
          padding: 6px 12px;
          font-size: 14px;
          font-weight: 400;
          line-height: 1.5;
          color: #495057;
          vertical-align: middle;
          background: #fff url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' width='4' height='5' viewBox='0 0 4 5'%3e%3cpath fill='%23343a40' d='M2 0L0 2h4zm0 5L0 3h4z'/%3e%3c/svg%3e") right .75rem center/8px 10px no-repeat;
          border-radius: 5px;
          -webkit-appearance: none;
          -moz-appearance: none;
          appearance: none;
          border: 1px solid #ced4da;
          font-family: 'Inter', sans-serif;
          letter-spacing: 0;
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc .toast-body  button.add-to-cart-btn {
          background-color: #28a745;
          border: 0;
          color: #fff;
          width: 100%;
          display: flex;
          align-items: center;
          justify-content: center;
          gap: 0px;
          margin-top: 5px;
          padding: 0 0;
          font-family: 'Inter', sans-serif;
          font-weight: 600;
          letter-spacing: 0;
          border-radius: 5px;
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc .toast-body button.add-to-cart-btn span.add-to-cart-btn-icon svg.icon.icon-cart {
          width: 34px;
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc  .toast-body {
          padding: 12px;
          background-color: rgb(255 255 255 / 90%);
      }
      div#product-sticky-atc-wrapper div#product-sticky-atc .toast-body .d-flex.align-items-center.mb-0 {
          display: flex;
          align-items: center;
      }
      @media (max-width:991px) {
        .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item {
            width: 48%;
            max-width: 50%;
        }
        .cstm-product-slider-component .cstm-prod-slide-component-ul {
            display: flex;
            gap: 26px;
        }
        .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item .multicolumn-card__image-wrapper {
            width: 32px;
            height: 32px;
        }
      }
      @media (max-width: 767px) {
        .cstm-product-slider-component .cstm-prod-slide-component-ul {
            padding: 0 1rem;
        }
        .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item {
            width: 100%;
            max-width: 100%;
        }
        .cstm-product-slider-component .cstm-prod-slide-component-ul {
            gap: 22px;
        }
        .cstm-product-slider-component .cstm-prod-slide-component-ul .multicolumn-list__item .multicolumn-card__image-wrapper {
            width: 30px;
            height: 30px;
        }
        div#product-sticky-atc-wrapper div#product-sticky-atc {
            min-width: unset;
            max-width: unset;
            width: calc(100% - 30px);
        }
      }
    </style>
    <!-- 21-04-2024 Product Page Section Css End -->
  </head>

  <body class="gradient{% if settings.animations_hover_elements != 'none' %} animate--hover-{{ settings.animations_hover_elements }}{% endif %}">
    <a class="skip-to-content-link button visually-hidden" href="#MainContent">
      {{ 'accessibility.skip_to_text' | t }}
    </a>

    {%- if settings.cart_type == 'drawer' -%}
      {%- render 'cart-drawer' -%}
    {%- endif -%}

    {% sections 'header-group' %}

    <main id="MainContent" class="content-for-layout focus-none" role="main" tabindex="-1">
      {{ content_for_layout }}
    </main>

{% if template.name == 'product' %}
    
<section id="shopify-section-template--16742497484955__multicolumn_9DKDTe" class="shopify-section section"><link href="//andrii-store-test.myshopify.com/cdn/shop/t/7/assets/section-multicolumn.css?v=120651070842298201681711282261" rel="stylesheet" type="text/css" media="all">
<link href="//andrii-store-test.myshopify.com/cdn/shop/t/7/assets/component-slider.css?v=142503135496229589681711282261" rel="stylesheet" type="text/css" media="all">
<style data-shopify="">.section-template--16742497484955__multicolumn_9DKDTe-padding {
    padding-top: 27px;
    padding-bottom: 27px;
  }

  @media screen and (min-width: 750px) {
    .section-template--16742497484955__multicolumn_9DKDTe-padding {
      padding-top: 36px;
      padding-bottom: 36px;
    }
  }</style><div class="multicolumn color-background-1 gradient background-primary no-heading">
  <div class="section-template--16742497484955__multicolumn_9DKDTe-padding isolate scroll-trigger animate--slide-in" data-cascade="" style="--animation-order: 0;"><slider-component class="slider-mobile-gutter cstm-product-slider-component">
      <ul class="cstm-prod-slide-component-ul multicolumn-list contains-content-container grid grid--1-col-tablet-down grid--4-col-desktop" id="Slider-template--16742497484955__multicolumn_9DKDTe" role="list"><li id="Slide-template--16742497484955__multicolumn_9DKDTe-1" class="multicolumn-list__item grid__item scroll-trigger animate--slide-in" data-cascade="" style="--animation-order: 0;">
            <div class="multicolumn-card content-container">
                <div class="multicolumn-card__image-wrapper multicolumn-card__image-wrapper--third-width multicolumn-card-spacing">
                  <div class="media media--transparent media--square">
                    <img src=https://cdn.shopify.com/s/files/1/0620/6514/8059/files/123.svg?v=1713354570
                    " class="multicolumn-card__image">
                  </div>
                </div><div class="multicolumn-card__info"><h3 class="inline-richtext">Free shipping</h3><div class="rte"><p>our shipping is free worldwide.</p><p></p></div></div>
            </div>
          </li><li id="Slide-template--16742497484955__multicolumn_9DKDTe-2" class="multicolumn-list__item grid__item scroll-trigger animate--slide-in" data-cascade="" style="--animation-order: 1;">
            <div class="multicolumn-card content-container">
                <div class="multicolumn-card__image-wrapper multicolumn-card__image-wrapper--third-width multicolumn-card-spacing">
                  <div class="media media--transparent media--square">
                    <img src=https://cdn.shopify.com/s/files/1/0620/6514/8059/files/abcd_svg.svg?v=1713354408
                    " class="multicolumn-card__image">
                  </div>
                </div><div class="multicolumn-card__info"><h3 class="inline-richtext">Guaranteed Purchase</h3><div class="rte"><p>Store registered and with SSL certificate.</p></div></div>
            </div>
          </li><li id="Slide-template--16742497484955__multicolumn_9DKDTe-3" class="multicolumn-list__item grid__item scroll-trigger animate--slide-in" data-cascade="" style="--animation-order: 2;">
            <div class="multicolumn-card content-container">
                <div class="multicolumn-card__image-wrapper multicolumn-card__image-wrapper--third-width multicolumn-card-spacing">
                  <div class="media media--transparent media--square">
                    <img src=https://cdn.shopify.com/s/files/1/0620/6514/8059/files/abcd1_2.svg?v=1713354402
                    " class="multicolumn-card__image">
                  </div>
                </div><div class="multicolumn-card__info"><h3 class="inline-richtext">Secure Payment</h3><div class="rte"><p>Strongly secure environment for payments.</p></div></div>
            </div>
          </li><li id="Slide-template--16742497484955__multicolumn_9DKDTe-4" class="multicolumn-list__item grid__item scroll-trigger animate--slide-in" data-cascade="" style="--animation-order: 3;">
            <div class="multicolumn-card content-container">
                <div class="multicolumn-card__image-wrapper multicolumn-card__image-wrapper--third-width multicolumn-card-spacing">
                  <div class="media media--transparent media--square">
                    <img src=https://cdn.shopify.com/s/files/1/0620/6514/8059/files/abcd1.svg?v=1713354390
                    " class="multicolumn-card__image">
                  </div>
                </div><div class="multicolumn-card__info"><h3 class="inline-richtext">Fast Support</h3><div class="rte"><p>Service from Monday to Friday </p><p>9am to 5pm.</p></div></div>
            </div>
          </li></ul></slider-component>
    <div class="center"></div>
  </div>
</div>


</section>


  {% else %}
  
{% endif %}
  

    {% sections 'footer-group' %}

    <ul hidden>
      <li id="a11y-refresh-page-message">{{ 'accessibility.refresh_page' | t }}</li>
      <li id="a11y-new-window-message">{{ 'accessibility.link_messages.new_window' | t }}</li>
    </ul>

    <script>
      window.shopUrl = '{{ request.origin }}';
      window.routes = {
        cart_add_url: '{{ routes.cart_add_url }}',
        cart_change_url: '{{ routes.cart_change_url }}',
        cart_update_url: '{{ routes.cart_update_url }}',
        cart_url: '{{ routes.cart_url }}',
        predictive_search_url: '{{ routes.predictive_search_url }}',
      };

      window.cartStrings = {
        error: `{{ 'sections.cart.cart_error' | t }}`,
        quantityError: `{{ 'sections.cart.cart_quantity_error_html' | t: quantity: '[quantity]' }}`,
      };

      window.variantStrings = {
        addToCart: `{{ 'products.product.add_to_cart' | t }}`,
        soldOut: `{{ 'products.product.sold_out' | t }}`,
        unavailable: `{{ 'products.product.unavailable' | t }}`,
        unavailable_with_option: `{{ 'products.product.value_unavailable' | t: option_value: '[value]' }}`,
      };

      window.quickOrderListStrings = {
        itemsAdded: `{{ 'sections.quick_order_list.items_added.other' | t: quantity: '[quantity]' }}`,
        itemAdded: `{{ 'sections.quick_order_list.items_added.one' | t: quantity: '[quantity]' }}`,
        itemsRemoved: `{{ 'sections.quick_order_list.items_removed.other' | t: quantity: '[quantity]' }}`,
        itemRemoved: `{{ 'sections.quick_order_list.items_removed.one' | t: quantity: '[quantity]' }}`,
        viewCart: `{{- 'sections.quick_order_list.view_cart' | t -}}`,
        each: `{{- 'sections.quick_order_list.each' | t: money: '[money]' }}`,
      };

      window.accessibilityStrings = {
        imageAvailable: `{{ 'products.product.media.image_available' | t: index: '[index]' }}`,
        shareSuccess: `{{ 'general.share.success_message' | t }}`,
        pauseSlideshow: `{{ 'sections.slideshow.pause_slideshow' | t }}`,
        playSlideshow: `{{ 'sections.slideshow.play_slideshow' | t }}`,
        recipientFormExpanded: `{{ 'recipient.form.expanded' | t }}`,
        recipientFormCollapsed: `{{ 'recipient.form.collapsed' | t }}`,
      };
    </script>

    {%- if settings.predictive_search_enabled -%}
      <script src="{{ 'predictive-search.js' | asset_url }}" defer="defer"></script>
    {%- endif -%}
    <style>
                <!-- changes @r -->
            section#shopify-section-template--16248000741531__1707804017e9a172b3 {
                background-color: #fff;
            }

              product-recommendations.related-products.page-width.section-template--16248000741531__related-products-padding.isolate.scroll-trigger.animate--slide-in.product-recommendations--loaded {
                padding-bottom: 70px;
            }

              .card__information {
                margin-bottom: -40px;
            }
              .jdgm-histogram__bar-content {
                background-color: yellow;
            }
              /* section#shopify-section-template--16248000741531__1707804017e9a172b3 {
                background: #fff;
            } */



      @media(min-width: 768px){
         .thumbnail-list.slider--tablet-up .thumbnail-list__item.slider__slide {
                width: 100% !important;
            }
      }
             



            .thumbnail-slider .thumbnail-list.slider--tablet-up {
                flex-direction: column !important;
            }

            .product--thumbnail_slider .thumbnail-slider {
                display: flex;
                align-items: center;
                width: 20% !important;
            }

            slider-component#GalleryViewer-template--16248000741531__main {
                width: 80% !important;
            }

            .grid__item.product__media-wrapper media-gallery {
                flex-direction: row-reverse !important;
                display: flex !important;
            }

            section#shopify-section-template--16248000741531__1707804017e9a172b3 {
                background: #fff9e8 !important;
                color: #000 !important;
            }

            .product-form__input input[type=radio]+label {
                padding: 2rem 2rem !important;
            }

             h2.jdgm-rev-widg__title {
                color: #000;
            }
              img.reviewstar {
                width: 35%;
            }
            slider-component {
          width: 80%;
      }


      @media(max-width: 767px){
  .grid__item.product__media-wrapper media-gallery{
    flex-direction: column !important;
  }
  .thumbnail-slider .thumbnail-list.slider--tablet-up{
    flex-direction: row !important;
  }
        .product--thumbnail_slider .thumbnail-slider{
          width: 100% !important;
        }
}

    </style>

    <script>
      $(document).ready(function(){
        $('.product__accordion').each(function(index, item){
          if (index === 0){
            $(this).find('details').attr('open', '');
            $(this).find('summary').attr('aria-expanded','true');
          }
        });
      });
    </script>

    <!--
      <script>
        $(document).ready(function(){
          $('.testimonial-slider').slick({
            dots: true,
            infinite: true,
            speed: 300,
            slidesToShow: 3,
            adaptiveHeight: true
          });
        });

      </script>
    -->
     <style>
                              /* featured-collection @r */
                              .testimonial .heading {
                            text-align: center;
                        }

                              .testimonial .row {
                          overflow: hidden;
                        }

                        .testimonial .col-4 {
                          float: left;
                          width: 33.333%;
                          box-sizing: border-box;
                        }
                              section#shopify-section-template--16248000577691__featured_collection_469WEn .quick-add {
                            margin-bottom: -40px;
                        }
                              section#shopify-section-template--16248000577691__featured_collection_469WEn .collection__view-all {
                            margin-top: 55px;
                        }
                        .slider-mobile-gutter li {
                            background: #9e9e9e17;
                            padding: 10px;
                        }

                           section#shopify-section-template--16248000577691__featured_collection_469WEn .quick-add {
                            margin-bottom: -40px;
                            margin-left: -10px !important;
                            margin-right: -10px;
                        }
                           div#shopify-section-template--16248000544923__product-grid {
                            margin-bottom: 30px;
                        }
                        section#shopify-section-template--16248000741531__main .product__title h1 {
                      font-size: 25px;
                  }
                  .rowimage {
                display: flex;
            }
                  .rowimage .img img {
                width: 50%;
            }
            .section-template--16248000544923__product-grid-padding .card-information {
          padding-bottom: 0 !important;
          bottom: 0;
          position: fixed;
      }
       .highlighted-slide {
          height: 390px !important;
          margin-bottom: 48px !important;
          margin-top: -48px !important;
      }
            .slick-track {
          padding-top: 5% !important;
      }

            .card--standard>.card__content .card__information {
          display: flex !important;
          flex-direction: column;
          justify-content: space-between;
      }

            .custom_collection_title_view {
          display: flex;
          flex-direction: row;
          justify-content: space-between;
          align-items: flex-end;
      }
            .card__information h3 a {
      	font-size: 17px;
      }
            li.grid__item.scroll-trigger.animate--slide-in {
          margin-bottom: 6%;
      }

      /* 10-04-2024 Css Start */
      .bottom-footer-content {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    column-gap: 75px;
        width: 100%;
}
      .bottom-footer-content .cstm-payment-box {
    max-width: 100%;
    width: 100%;
}
.bottom-footer-content .cstm-payment-box .payment-methods-div {
    position: relative;
    display: grid;
    grid-template-columns: repeat(6, 14fr);
    gap: 10px;
}
      .bottom-footer-content .cstm-payment-box .payment-methods-div img {
    width: 100%;
    height: 34px;
    object-fit: contain;
}
      .bottom-footer-content .cstm-payment-box p.Footer-payment-headign {
    text-align: center;
    color: #fff;
}
      .bottom-footer-content .cstm-payment-box .safety-methods-div {
    position: relative;
    display: flex;
    grid-template-columns: repeat(2, 1fr);
    gap: 12px;
}
.bottom-footer-content .cstm-payment-box .safety-methods-div img {
    width: 100%;
    height: 68px;
    object-fit: contain;
}
      .social-media-div {
    text-align: center;
}
.social-media-div img {
    width: 28px;
    filter: brightness(150) invert(1);
    height: 28px;
    object-fit: contain;
    margin-inline: 5px;
}
      .payment-icon {
    display: flex;
}
      .payment-icon svg {
    width: 100%;
    height: auto;
}

    @media (max-width:991px) {
      .bottom-footer-content {
    column-gap: 42px;
}
    }

          @media (max-width:767px) {
      .bottom-footer-content {
    row-gap: 30px;
        grid-template-columns: repeat(1, 14fr);
}
            .bottom-footer-content .cstm-payment-box .safety-methods-div img {
    width: 100%;
    height: 55px;
    object-fit: contain;
}
    }

.multicolumn-card.content-container {
    display: flex;
    align-items: center;
}

      slider-component {
	width: 100%;
}
      

      .multicolumn-list__item {
	background: transparent !important;
}

.multicolumn.background-primary .multicolumn-card {
    background: transparent !important;
}
      
      /* 10-04-2024 Css End */

       /* css sticky cart css */
       button.add-to-cart-btn.btn.btn-sm.button-buy.btn-block.d-flex.align-items-center.justify-content-center.animate__animated:hover {
    background-color: #000 !important;
    cursor: pointer;
}
       /* product title */
.product__title h1 {
    font-size: 17px;
    font-weight: bold;
}
      
       /* shake animation style start */

@keyframes shake {
  0% { transform: translateX(0); }
  25% { transform: translateX(-20px); }
  50% { transform: translateX(20px); }
  75% { transform: translateX(-20px); }
  100% { transform: translateX(0); }
}

.shakeAnimation.shake {
  animation: shake 0.5s infinite;
}
       /* shake animation style End */
    
       
     </style>

    <script>
      $(document).ready(function() {

    $('.buy-now-btn').on('click', function(e) {
        e.preventDefault();

        var variantId = $(this).data('variant-id'); // Get the variant ID from data attribute
        var quantity = $(this).data('quantity') || 1; // Set default quantity to 1 if not provided

        // Add the product to the cart using AJAX
        jQuery.post('/cart/add.js', {
            quantity: quantity,
            id: variantId
        }).done(function() {
            // Redirect to the checkout page
            window.location.href = '/checkout';
        });
    });
});

    
/////// shake animation JS Start //////
      
      document.addEventListener("DOMContentLoaded", function() {
  const shakeAnimationDiv = document.getElementById('shakeAnimation');

  function startAnimation() {
    shakeAnimationDiv.classList.add('shake');
  }
        
  function stopAnimation() {
    shakeAnimationDiv.classList.remove('shake');
  }

  function startAnimationLoop() {
    setTimeout(function() {
      startAnimation();
      setTimeout(stopAnimation, 2000);
      setTimeout(startAnimationLoop, 5000);
    }, 2000);
  }
  startAnimationLoop();
});
      
/////// shake animation JS end //////
    

        
    $('.close').on('click', function(e) {
$('#product-sticky-atc').remove();
    });
      
    </script>


    <style>

.or_better_design {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: flex-end;
}

      .product-form__quantity {
    max-width: fit-content;
    margin-right: 10px;
}

      .product-form__quantity {
    max-width: fit-content;
    margin-right: 10px;
    margin-bottom: 10px;
}

      .main-font-title {
    padding-top: 0 !important;
}
      
    </style>


    
  </body>
</html>
