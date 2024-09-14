---
layout: default
---
 <ul>
    <li>
       <p>platform tech lead @ <a href="https://mercedes-benz.io">mercedes-benz.io</a></p>
    </li>
    <li>
      <a href="{{ 'https://github.com/' | append: site.theme_config.github_username }}">github</a> /
      <a href="{{ 'https://linkedin.com/in/' | append: site.theme_config.linkedin_username }}">linkedin</a> /
      <a href="{{ 'https://twitter.com/' | append: site.theme_config.twitter_username }}">twitter</a>
    </li>
    <li>
      find me: luis @ `{{ site.theme_config.base_domain_url }}`
    </li>
 </ul>
 <ul>
    <li><b>materialization of random thougs</b>:</li>
    <ul class="post-list">
      {% for post in site.posts %}
        <li>
          <a class="post-list" href="{{ post.url }}">{{ post.title }}</a> (<span class="post-meta">{{ post.date | date: site.theme_config.date_format }}</span>)
        </li>
      {% endfor %}
    </ul>
 </ul>
 <ul>
    <li>
       my experiments:
       <ul>
          <li>
            <a href="https://github.com/lmmendes/game-boy-opcodes" target="_blank">game-boy-opcodes</a>: game boy cpu (sharp LR35902) <a href="https://gb.insertcoin.dev/">instruction set (opcodes)</a>
          </li>
          <!--
          <li>
            <a href="https://github.com/lmmendes/chip8-rb" target="_blank">
            chip8-rb</a>: A chip 8 interpreter in ruby (wip)</li>
          <li><a href="https://github.com/lmmendes/okada" target="_blank">okada</a>: A work in progress to try to implement a Game Boy emulator in Ruby.</li>
          -->
       </ul>
    </li>
    <li>
       other interests:
       <ul>
          <li><a href="https://www.goodreads.com/lmmendes">reading</a></li>
          <li>emuladors</li>
          <li>retro computing</li>
          <li>algorithms and data structures</li>
       </ul>
    </li>
 </ul>
