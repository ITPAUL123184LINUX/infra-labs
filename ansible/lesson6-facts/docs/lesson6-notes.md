<h1>Lesson 6 – Working with Facts (Deep Notes)</h1>

<h2>Goal</h2>
<b>Learn how Ansible gathers, uses, and defines system information (“facts”) to build dynamic automation.</b>

<h3>Control node</h3>
<ul>
  <li><b>Host:</b> <code>control</code> (where Ansible commands run)</li>
</ul>

<h3>Managed nodes</h3>
<ul>
  <li><b>Hosts:</b> <code>ansible1</code>, <code>ansible2</code>, <code>ansible3</code> — reachable via SSH as user <b>ansible</b>.</li>
</ul>

<hr/>

<h2>6.1 Understanding Facts</h2>
<p>
Ansible <b>facts</b> are automatically gathered variables that describe a host’s system information.  
They’re discovered when a play begins using the <code>setup</code> module.
</p>

<ul>
  <li>Facts contain host data like memory, IP, OS, and hardware details.</li>
  <li>Stored in a dictionary called <code>ansible_facts</code>.</li>
  <li>Useful for making plays dynamic (no hardcoded values).</li>
  <li>Examples: <code>ansible_facts.default_ipv4.address</code>, <code>ansible_facts.memtotal_mb</code>, <code>ansible_facts.distribution</code>.</li>
</ul>

<p><b>Example:</b></p>
<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.1ipfacts.yml.PNG"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.1ipfacts-output.PNG">
</p>

<hr/>

<h2>6.2 Managing Fact Gathering</h2>

<ul>
  <li>By default, Ansible gathers facts automatically before running tasks.</li>
  <li>Disable it to speed things up:
    <pre><code>gather_facts: no</code></pre>
  </li>
  <li>Even if disabled, you can re-gather with the <code>setup</code> module:
    <pre><code>ansible all -m ansible.builtin.setup -a 'filter=ansible_hostname' -o</code></pre>
  </li>
  <li>View all host data with:
    <pre><code>ansible all -m ansible.builtin.setup | head -n 25</code></pre>
  </li>
  <li>Facts follow a dotted hierarchy — like <code>ansible_facts.default_ipv4.address</code> — meaning you can drill into nested data.</li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.1P2.PNG" width="900">
</p>

<p><b>Tip:</b> Use <code>debug</code> to display any single fact for clarity:</p>
<pre><code>- debug:
    msg: "OS family is {{ ansible_facts.os_family }}"
</code></pre>

<hr/>

<h2>6.3 Troubleshooting Slow Fact Collection</h2>

<ul>
  <li>Fact gathering can slow down large environments or misconfigured networks.</li>
  <li>Solutions:
    <ul>
      <li>Enable <b>fact caching</b> to reuse facts between runs.</li>
      <li>Ensure proper <b>hostname resolution</b> — each host must resolve the others.</li>
      <li>Maintain consistent <code>/etc/hosts</code> entries across all managed nodes.</li>
    </ul>
  </li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.1P3.PNG" width="900">
</p>

<p><b>Exam Insight:</b> If your plays take too long on “Gathering Facts,” check SSH connectivity and name resolution first.</p>

<hr/>

<h2>6.4 Using Multi-Tier Variables</h2>
<p>Ansible merges variables from multiple levels (global → group → host → task). Facts live near the top in precedence order.</p>
<ul>
  <li>Playbook vars override gathered facts only temporarily.</li>
  <li>Fact caching can persist facts between runs (if configured).</li>
</ul>

<hr/>

<h2>6.5 Dictionaries and Arrays (Deep Dive)</h2>
<p>
Multi-valued variables in Ansible can be expressed as <b>lists</b> (arrays) or <b>dictionaries</b> (hashes).  
Facts themselves are stored as dictionaries, while lists are often used for loops.
</p>

<ul>
  <li><b>Dictionary:</b> An unordered key-value structure (<code>{ }</code>) — like <code>ansible_facts</code>.</li>
  <li><b>Array/List:</b> An ordered collection (<code>[ ]</code>) — often used with <code>loop:</code> or <code>with_items:</code>.</li>
  <li><b>Strings:</b> Plain values wrapped in quotes (<code>" "</code>).</li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.3.PNG" width="800"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.3p2.PNG" width="800"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.3p3.PNG" width="800"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.3p4.PNG" width="800"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.3p5.PNG" width="800"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.3p6.PNG" width="800"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.3p7.PNG" width="800">
</p>

<p><b>Dictionary example:</b></p>
<pre><code>users:
  linda:
    username: linda
    shell: /bin/bash
  lisa:
    username: lisa
    shell: /bin/sh
</code></pre>

<p>or equivalently:</p>
<pre><code>users:
  linda: { username: 'linda', shell: '/bin/bash' }
  lisa:  { username: 'lisa',  shell: '/bin/sh' }
</code></pre>

<p><b>How to access dictionary values:</b></p>
<pre><code>{{ users['linda']['shell'] }}
{{ users.linda.shell }}
</code></pre>

<p><b>Recognizing structures in output:</b></p>
<ul>
  <li><code>[ ]</code> → List (array)</li>
  <li><code>{ }</code> → Dictionary (key-value pairs)</li>
  <li><code>" "</code> → String</li>
</ul>

<hr/>

<h2>Lesson 6 Lab – Dictionaries vs Arrays</h2>

<p>Below, two playbooks demonstrate the difference between arrays (lists) and dictionaries (hashes):</p>

<h3>Dictionary Example</h3>
<p><b>Playbook:</b></p>
<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/multi-dictionary.yml.PNG" width="700"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/multi-dictionary-output.PNG" width="900">
</p>

<h3>Array Example</h3>
<p><b>Playbook:</b></p>
<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/multi-list.yml.PNG" width="700"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/multi-list.ymloutpt.PNG" width="900">
</p>

<p><b>Output Summary:</b></p>
<ul>
  <li>Lists are perfect for looping through multiple objects.</li>
  <li>Dictionaries are best when you want structured, named data (like host facts).</li>
</ul>

<hr/>

<h2>6.6 Defining Custom Facts</h2>
<ul>
  <li>Create files under <code>/etc/ansible/facts.d/</code> on managed nodes.</li>
  <li>Each file must end with <code>.fact</code> and contain INI or JSON data.</li>
  <li>Gathered automatically during <code>setup</code>.</li>
</ul>

<pre><code>[localinfo]
site=datacenter1
tier=web
</code></pre>

<pre><code>ansible all -m ansible.builtin.setup -a 'filter=ansible_local' -o</co_
