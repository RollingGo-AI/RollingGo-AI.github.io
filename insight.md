---
layout: page
title: RollingGo Insight
permalink: /insight/
---

<style>
  .page-heading { display: none; }
  .covers {
    display: flex;
    justify-content: center;
    gap: 24px;
    margin: 20px 0 40px;
  }

  .cover-card {
    width: 320px;
    flex-shrink: 0;
    border-radius: 6px;
    overflow: hidden;
    box-shadow: 0 8px 32px rgba(0,0,0,0.15);
    transition: transform 0.3s ease;
  }

  .cover-card:hover {
    transform: translateY(-4px);
  }

  .cover-img {
    width: 320px;
    height: 400px;
    position: relative;
    overflow: hidden;
  }

  .cover-img .bg {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
  }

  .cover-img .overlay {
    position: absolute;
    inset: 0;
    background: linear-gradient(
      180deg,
      rgba(0,0,0,0.15) 0%,
      rgba(0,0,0,0) 25%,
      rgba(0,0,0,0) 50%,
      rgba(0,0,0,0.5) 75%,
      rgba(0,0,0,0.82) 100%
    );
  }

  .cover-img .top-bar {
    position: absolute;
    top: 0; left: 0; right: 0;
    padding: 20px 22px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .cover-img .logo {
    font-family: 'Playfair Display', serif;
    font-size: 13px;
    font-weight: 900;
    color: #fff;
    letter-spacing: 4px;
    text-transform: uppercase;
  }

  .cover-img .issue {
    font-size: 8px;
    color: rgba(255,255,255,0.7);
    letter-spacing: 2px;
    text-transform: uppercase;
    font-weight: 300;
  }

  .cover-img .main-text {
    position: absolute;
    bottom: 54px;
    left: 22px;
    right: 22px;
  }

  .cover-img .main-text h2 {
    font-family: 'Playfair Display', serif;
    font-size: 20px;
    font-weight: 700;
    color: #fff;
    line-height: 1.3;
    text-shadow: 0 2px 20px rgba(0,0,0,0.4);
  }

  .cover-img .main-text h2 em {
    font-style: italic;
    color: #f0c27f;
  }

  .cover-img .divider {
    width: 32px;
    height: 2px;
    background: #f0c27f;
    margin: 12px 0;
  }

  .cover-img .subtitle {
    font-size: 9px;
    color: rgba(255,255,255,0.8);
    font-weight: 300;
    letter-spacing: 0.3px;
    line-height: 1.6;
    max-width: 240px;
  }

  .cover-img .bottom-bar {
    position: absolute;
    bottom: 18px;
    left: 22px;
    right: 22px;
    display: flex;
    justify-content: space-between;
  }

  .cover-img .brand-tag {
    font-size: 7px;
    color: rgba(255,255,255,0.45);
    letter-spacing: 2px;
    text-transform: uppercase;
    font-weight: 600;
  }

  .cover-img .category {
    font-size: 7px;
    color: rgba(255,255,255,0.35);
    letter-spacing: 1.5px;
    text-transform: uppercase;
  }

  .label {
    text-align: center;
    margin-top: 14px;
  }

  .label span {
    font-size: 10px;
    color: rgba(0,0,0,0.35);
    letter-spacing: 2px;
    text-transform: uppercase;
  }

  .placeholder {
    width: 320px;
    height: 400px;
    border: 1px dashed #ccc;
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .placeholder span {
    font-size: 11px;
    color: #aaa;
    letter-spacing: 2px;
    text-transform: uppercase;
  }

  .plain-card {
    width: 320px;
    height: 400px;
    border-radius: 6px;
    overflow: hidden;
    box-shadow: 0 8px 32px rgba(0,0,0,0.15);
    transition: transform 0.3s ease;
  }

  .plain-card:hover {
    transform: translateY(-4px);
  }

  .plain-card img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
  }
</style>

<div class="covers">
  <div>
    <div class="cover-card">
      <div class="cover-img">
        <img class="bg" src="{{ '/assets/images/Lake-Como-Hotel.jpg' | relative_url }}" alt="Cover 1">
        <div class="overlay"></div>
        <div class="top-bar">
          <div class="logo">RollingGo</div>
          <div class="issue">Issue 01 · 2026</div>
        </div>
        <div class="main-text">
          <h2>The most <em>exhausting</em> part of travel is <em>booking.</em></h2>
          <div class="divider"></div>
          <div class="subtitle">Not the journey. Not the destination. The endless tabs, the comparisons, the mental load before you even leave.</div>
        </div>
        <div class="bottom-bar">
          <div class="brand-tag">RollingGo · Travel Reimagined</div>
          <div class="category">AI × Travel Insight</div>
        </div>
      </div>
    </div>
    <div class="label"><span>Issue 01</span></div>
  </div>

  <div>
    <div class="plain-card">
      <img src="{{ '/assets/images/layla_travel.png' | relative_url }}" alt="Layla Travel">
    </div>
    <div class="label"><span>Issue 02</span></div>
  </div>

  <div>
    <div class="placeholder"><span>Coming Soon</span></div>
    <div class="label"><span>Issue 03</span></div>
  </div>
</div>