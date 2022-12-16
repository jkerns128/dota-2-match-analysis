# Dota 2: What hero is the most toxic?

## Introduction

Dota 2 is a 5v5 MOBA(Multiplayer Online Battle Arena) where both teams work together to defeat the other team. Each team selects 5 heroes for their players to play, each with unique abilities and quirks. Heroes can also be augmented by leveling up and buying items. These items can have many different effects, ranging from resetting the cooldowns of all abilities to reviving your hero immediately after death. An important part of the game is being able to kill the enemy heroes, which grants gold for item purchases and experience for abiliy upgrades. 
 
A key part of the game features a voice chat system, where players can talk to their teammates to coordinate and cooperate. There is also a text chat system which is also used for the same goals. Some players out of frustration may use these means to insult or belittle their enemies, often players will do this to their fellow teamates. While the players who do this are known as "toxic" in their communities, it is still very common to find players behaving like this in games. Valve has implemented some systems, like behavior score, in order to seperate out these players from more friendly ones, but it does not remove these players from the game. In my analysis, I chose to look at why some players may be more toxic than other players.

## Data Exploration
 
<body class="jp-Notebook" data-jp-theme-light="true" data-jp-theme-name="JupyterLab Light">
<div class="jp-Cell jp-CodeCell jp-Notebook-cell   ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[1]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="c1">#data from opendota</span>
<span class="kn">import</span> <span class="nn">pandas</span> <span class="k">as</span> <span class="nn">pd</span>
<span class="kn">import</span> <span class="nn">statsmodels.formula.api</span> <span class="k">as</span> <span class="nn">smf</span>
<span class="kn">import</span> <span class="nn">numpy</span> <span class="k">as</span> <span class="nn">np</span>
<span class="kn">import</span> <span class="nn">matplotlib.pyplot</span> <span class="k">as</span> <span class="nn">plt</span>
<span class="kn">from</span> <span class="nn">matplotlib.lines</span> <span class="kn">import</span> <span class="n">Line2D</span> <span class="c1"># for the legend</span>
<span class="kn">import</span> <span class="nn">pylab</span> <span class="k">as</span> <span class="nn">pl</span>
<span class="kn">import</span> <span class="nn">random</span>
<span class="kn">import</span> <span class="nn">string</span>
<span class="kn">import</span> <span class="nn">math</span>
<span class="kn">from</span> <span class="nn">sklearn.linear_model</span> <span class="kn">import</span> <span class="n">LinearRegression</span>
<span class="kn">from</span> <span class="nn">sklearn</span> <span class="kn">import</span> <span class="n">datasets</span><span class="p">,</span> <span class="n">linear_model</span>
<span class="n">pd</span><span class="o">.</span><span class="n">set_option</span><span class="p">(</span><span class="s1">&#39;display.max_columns&#39;</span><span class="p">,</span> <span class="mi">500</span><span class="p">)</span>
<span class="n">pd</span><span class="o">.</span><span class="n">set_option</span><span class="p">(</span><span class="s1">&#39;display.max_rows&#39;</span><span class="p">,</span> <span class="mi">130</span><span class="p">)</span>

<span class="n">match_data</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">read_csv</span><span class="p">(</span><span class="s1">&#39;match.csv&#39;</span><span class="p">)</span>
<span class="n">players_data</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">read_csv</span><span class="p">(</span><span class="s1">&#39;players.csv&#39;</span><span class="p">)</span>
<span class="n">hero_names</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">read_csv</span><span class="p">(</span><span class="s1">&#39;hero_names.csv&#39;</span><span class="p">)</span>
<span class="n">item_ids</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">read_csv</span><span class="p">(</span><span class="s1">&#39;item_ids.csv&#39;</span><span class="p">)</span>
<span class="n">chat</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">read_csv</span><span class="p">(</span><span class="s1">&#39;chat.csv&#39;</span><span class="p">)</span>
</pre></div>

</div>
</div>
</div>
</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell   ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[2]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="c1"># I want to make the player_slot to match between all tables, so I will change it to a standard</span>
<span class="c1"># No matches were played on arc warden in my dataset</span>
<span class="k">def</span> <span class="nf">pslot</span><span class="p">(</span><span class="n">x</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">x</span> <span class="o">&gt;=</span> <span class="mi">128</span><span class="p">:</span>
        <span class="k">return</span> <span class="n">x</span><span class="o">-</span><span class="mi">123</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="k">return</span> <span class="n">x</span>

<span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;player_slot&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;player_slot&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">apply</span><span class="p">(</span><span class="n">pslot</span><span class="p">)</span>
<span class="n">players_data</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">merge</span><span class="p">(</span><span class="n">players_data</span><span class="p">,</span> <span class="n">match_data</span><span class="p">,</span> <span class="n">on</span><span class="o">=</span><span class="s1">&#39;match_id&#39;</span><span class="p">)</span>
<span class="n">players_data</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">merge</span><span class="p">(</span><span class="n">players_data</span><span class="p">,</span> <span class="n">hero_names</span><span class="p">[[</span><span class="s1">&#39;hero_id&#39;</span><span class="p">,</span><span class="s1">&#39;localized_name&#39;</span><span class="p">]],</span> <span class="n">on</span><span class="o">=</span><span class="s1">&#39;hero_id&#39;</span><span class="p">)</span>
<span class="n">players_data</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[[</span><span class="s1">&#39;match_id&#39;</span><span class="p">,</span><span class="s1">&#39;hero_id&#39;</span><span class="p">,</span><span class="s1">&#39;player_slot&#39;</span><span class="p">,</span><span class="s1">&#39;kills&#39;</span><span class="p">,</span><span class="s1">&#39;deaths&#39;</span><span class="p">,</span><span class="s1">&#39;assists&#39;</span><span class="p">,</span><span class="s1">&#39;item_0&#39;</span><span class="p">,</span><span class="s1">&#39;item_1&#39;</span><span class="p">,</span><span class="s1">&#39;item_2&#39;</span><span class="p">,</span><span class="s1">&#39;item_3&#39;</span><span class="p">,</span><span class="s1">&#39;item_4&#39;</span><span class="p">,</span><span class="s1">&#39;item_5&#39;</span><span class="p">,</span><span class="s1">&#39;radiant_win&#39;</span><span class="p">,</span><span class="s1">&#39;localized_name&#39;</span><span class="p">]]</span>
<span class="n">players_data</span>
</pre></div>

     </div>
</div>
</div>
</div>

<div class="jp-Cell-outputWrapper">
<div class="jp-Collapser jp-OutputCollapser jp-Cell-outputCollapser">
</div>


<div class="jp-OutputArea jp-Cell-outputArea">

<div class="jp-OutputArea-child">

    
    <div class="jp-OutputPrompt jp-OutputArea-prompt">Out[2]:</div>



<div class="jp-RenderedHTMLCommon jp-RenderedHTML jp-OutputArea-output jp-OutputArea-executeResult" data-mime-type="text/html">
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>match_id</th>
      <th>hero_id</th>
      <th>player_slot</th>
      <th>kills</th>
      <th>deaths</th>
      <th>assists</th>
      <th>item_0</th>
      <th>item_1</th>
      <th>item_2</th>
      <th>item_3</th>
      <th>item_4</th>
      <th>item_5</th>
      <th>radiant_win</th>
      <th>localized_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>86</td>
      <td>0</td>
      <td>9</td>
      <td>3</td>
      <td>18</td>
      <td>180</td>
      <td>37</td>
      <td>73</td>
      <td>56</td>
      <td>108</td>
      <td>0</td>
      <td>True</td>
      <td>Rubick</td>
    </tr>
    <tr>
      <th>1</th>
      <td>9</td>
      <td>86</td>
      <td>3</td>
      <td>4</td>
      <td>14</td>
      <td>15</td>
      <td>1</td>
      <td>41</td>
      <td>180</td>
      <td>90</td>
      <td>43</td>
      <td>0</td>
      <td>False</td>
      <td>Rubick</td>
    </tr>
    <tr>
      <th>2</th>
      <td>16</td>
      <td>86</td>
      <td>1</td>
      <td>2</td>
      <td>4</td>
      <td>18</td>
      <td>180</td>
      <td>108</td>
      <td>102</td>
      <td>1</td>
      <td>58</td>
      <td>0</td>
      <td>False</td>
      <td>Rubick</td>
    </tr>
    <tr>
      <th>3</th>
      <td>28</td>
      <td>86</td>
      <td>0</td>
      <td>4</td>
      <td>7</td>
      <td>16</td>
      <td>214</td>
      <td>46</td>
      <td>100</td>
      <td>178</td>
      <td>0</td>
      <td>36</td>
      <td>True</td>
      <td>Rubick</td>
    </tr>
    <tr>
      <th>4</th>
      <td>37</td>
      <td>86</td>
      <td>2</td>
      <td>14</td>
      <td>7</td>
      <td>23</td>
      <td>48</td>
      <td>0</td>
      <td>108</td>
      <td>1</td>
      <td>96</td>
      <td>0</td>
      <td>True</td>
      <td>Rubick</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>499958</th>
      <td>49719</td>
      <td>66</td>
      <td>0</td>
      <td>0</td>
      <td>11</td>
      <td>2</td>
      <td>108</td>
      <td>94</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>False</td>
      <td>Chen</td>
    </tr>
    <tr>
      <th>499959</th>
      <td>49737</td>
      <td>66</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>False</td>
      <td>Chen</td>
    </tr>
    <tr>
      <th>499960</th>
      <td>49787</td>
      <td>66</td>
      <td>5</td>
      <td>2</td>
      <td>7</td>
      <td>4</td>
      <td>0</td>
      <td>46</td>
      <td>42</td>
      <td>108</td>
      <td>231</td>
      <td>88</td>
      <td>True</td>
      <td>Chen</td>
    </tr>
    <tr>
      <th>499961</th>
      <td>49837</td>
      <td>66</td>
      <td>8</td>
      <td>1</td>
      <td>8</td>
      <td>4</td>
      <td>180</td>
      <td>46</td>
      <td>79</td>
      <td>218</td>
      <td>88</td>
      <td>0</td>
      <td>True</td>
      <td>Chen</td>
    </tr>
    <tr>
      <th>499962</th>
      <td>49931</td>
      <td>66</td>
      <td>6</td>
      <td>4</td>
      <td>2</td>
      <td>12</td>
      <td>41</td>
      <td>79</td>
      <td>194</td>
      <td>0</td>
      <td>81</td>
      <td>180</td>
      <td>False</td>
      <td>Chen</td>
    </tr>
  </tbody>
</table>
<p>499963 rows × 14 columns</p>
</div>
</div>

</div>

</div>

</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell   ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[3]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="c1"># Finding the highest winrate heroes</span>
<span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;hero_won&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">players_data</span><span class="o">.</span><span class="n">apply</span><span class="p">((</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">hero_id</span> <span class="k">if</span> <span class="n">x</span><span class="o">.</span><span class="n">radiant_win</span> <span class="k">else</span> <span class="mi">0</span><span class="p">),</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
<span class="n">total_frequencies</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;hero_id&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">value_counts</span><span class="p">()</span>
<span class="n">win_frequencies</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;hero_won&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">value_counts</span><span class="p">()</span>
<span class="n">hero_play_df</span> <span class="o">=</span> <span class="n">win_frequencies</span><span class="o">.</span><span class="n">to_frame</span><span class="p">()</span><span class="o">.</span><span class="n">join</span><span class="p">(</span><span class="n">total_frequencies</span><span class="p">)</span>
<span class="n">hero_play_df</span> <span class="o">=</span> <span class="n">hero_play_df</span><span class="o">.</span><span class="n">drop</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
<span class="n">hero_play_df</span><span class="p">[</span><span class="s1">&#39;hero_winrate&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">hero_play_df</span><span class="p">[</span><span class="s1">&#39;hero_won&#39;</span><span class="p">]</span><span class="o">/</span><span class="n">hero_play_df</span><span class="p">[</span><span class="s1">&#39;hero_id&#39;</span><span class="p">]</span>
<span class="n">hero_play_df</span> <span class="o">=</span> <span class="n">hero_play_df</span><span class="o">.</span><span class="n">merge</span><span class="p">(</span><span class="n">hero_names</span><span class="p">,</span> <span class="n">left_index</span><span class="o">=</span><span class="kc">True</span><span class="p">,</span> <span class="n">right_on</span><span class="o">=</span><span class="s1">&#39;hero_id&#39;</span><span class="p">)</span>
<span class="n">hero_play_df</span> <span class="o">=</span> <span class="n">hero_play_df</span><span class="o">.</span><span class="n">drop</span><span class="p">(</span><span class="n">labels</span><span class="o">=</span><span class="p">[</span><span class="s1">&#39;name&#39;</span><span class="p">,</span><span class="s1">&#39;hero_id_y&#39;</span><span class="p">],</span><span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span><span class="o">.</span><span class="n">rename</span><span class="p">(</span><span class="n">columns</span><span class="o">=</span><span class="p">{</span><span class="s1">&#39;hero_id_x&#39;</span> <span class="p">:</span> <span class="s1">&#39;hero_plays&#39;</span><span class="p">})</span>
<span class="n">hero_play_df</span> <span class="o">=</span> <span class="n">hero_play_df</span><span class="o">.</span><span class="n">sort_values</span><span class="p">(</span><span class="n">by</span><span class="o">=</span><span class="s1">&#39;hero_winrate&#39;</span><span class="p">,</span> <span class="n">ascending</span><span class="o">=</span><span class="kc">False</span><span class="p">)</span>
<span class="n">hero_play_df</span>
</pre></div>

     </div>
</div>
</div>
</div>

<div class="jp-Cell-outputWrapper">
<div class="jp-Collapser jp-OutputCollapser jp-Cell-outputCollapser">
</div>


<div class="jp-OutputArea jp-Cell-outputArea">

<div class="jp-OutputArea-child">

    
    <div class="jp-OutputPrompt jp-OutputArea-prompt">Out[3]:</div>



<div class="jp-RenderedHTMLCommon jp-RenderedHTML jp-OutputArea-output jp-OutputArea-executeResult" data-mime-type="text/html">
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>hero_id</th>
      <th>hero_won</th>
      <th>hero_plays</th>
      <th>hero_winrate</th>
      <th>localized_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>46</th>
      <td>48</td>
      <td>1270</td>
      <td>2378.0</td>
      <td>0.534062</td>
      <td>Luna</td>
    </tr>
    <tr>
      <th>100</th>
      <td>102</td>
      <td>1764</td>
      <td>3310.0</td>
      <td>0.532931</td>
      <td>Abaddon</td>
    </tr>
    <tr>
      <th>25</th>
      <td>27</td>
      <td>1909</td>
      <td>3589.0</td>
      <td>0.531903</td>
      <td>Shadow Shaman</td>
    </tr>
    <tr>
      <th>14</th>
      <td>15</td>
      <td>1072</td>
      <td>2017.0</td>
      <td>0.531482</td>
      <td>Razor</td>
    </tr>
    <tr>
      <th>22</th>
      <td>23</td>
      <td>1348</td>
      <td>2543.0</td>
      <td>0.530083</td>
      <td>Kunkka</td>
    </tr>
    <tr>
      <th>102</th>
      <td>104</td>
      <td>4778</td>
      <td>9025.0</td>
      <td>0.529418</td>
      <td>Legion Commander</td>
    </tr>
    <tr>
      <th>61</th>
      <td>63</td>
      <td>1356</td>
      <td>2566.0</td>
      <td>0.528449</td>
      <td>Weaver</td>
    </tr>
    <tr>
      <th>35</th>
      <td>37</td>
      <td>988</td>
      <td>1871.0</td>
      <td>0.528060</td>
      <td>Warlock</td>
    </tr>
    <tr>
      <th>96</th>
      <td>98</td>
      <td>1548</td>
      <td>2934.0</td>
      <td>0.527607</td>
      <td>Timbersaw</td>
    </tr>
    <tr>
      <th>65</th>
      <td>67</td>
      <td>3513</td>
      <td>6660.0</td>
      <td>0.527477</td>
      <td>Spectre</td>
    </tr>
    <tr>
      <th>82</th>
      <td>84</td>
      <td>2296</td>
      <td>4353.0</td>
      <td>0.527452</td>
      <td>Ogre Magi</td>
    </tr>
    <tr>
      <th>104</th>
      <td>106</td>
      <td>3973</td>
      <td>7533.0</td>
      <td>0.527413</td>
      <td>Ember Spirit</td>
    </tr>
    <tr>
      <th>54</th>
      <td>56</td>
      <td>1307</td>
      <td>2479.0</td>
      <td>0.527229</td>
      <td>Clinkz</td>
    </tr>
    <tr>
      <th>17</th>
      <td>18</td>
      <td>1818</td>
      <td>3450.0</td>
      <td>0.526957</td>
      <td>Sven</td>
    </tr>
    <tr>
      <th>44</th>
      <td>46</td>
      <td>3183</td>
      <td>6042.0</td>
      <td>0.526812</td>
      <td>Templar Assassin</td>
    </tr>
    <tr>
      <th>89</th>
      <td>91</td>
      <td>892</td>
      <td>1694.0</td>
      <td>0.526564</td>
      <td>Io</td>
    </tr>
    <tr>
      <th>30</th>
      <td>32</td>
      <td>2179</td>
      <td>4140.0</td>
      <td>0.526329</td>
      <td>Riki</td>
    </tr>
    <tr>
      <th>73</th>
      <td>75</td>
      <td>3801</td>
      <td>7224.0</td>
      <td>0.526163</td>
      <td>Silencer</td>
    </tr>
    <tr>
      <th>52</th>
      <td>54</td>
      <td>1359</td>
      <td>2585.0</td>
      <td>0.525725</td>
      <td>Lifestealer</td>
    </tr>
    <tr>
      <th>21</th>
      <td>22</td>
      <td>2412</td>
      <td>4589.0</td>
      <td>0.525605</td>
      <td>Zeus</td>
    </tr>
    <tr>
      <th>34</th>
      <td>36</td>
      <td>3137</td>
      <td>5969.0</td>
      <td>0.525549</td>
      <td>Necrophos</td>
    </tr>
    <tr>
      <th>83</th>
      <td>85</td>
      <td>3123</td>
      <td>5951.0</td>
      <td>0.524786</td>
      <td>Undying</td>
    </tr>
    <tr>
      <th>37</th>
      <td>39</td>
      <td>5556</td>
      <td>10590.0</td>
      <td>0.524646</td>
      <td>Queen of Pain</td>
    </tr>
    <tr>
      <th>18</th>
      <td>19</td>
      <td>2783</td>
      <td>5305.0</td>
      <td>0.524599</td>
      <td>Tiny</td>
    </tr>
    <tr>
      <th>98</th>
      <td>100</td>
      <td>5406</td>
      <td>10306.0</td>
      <td>0.524549</td>
      <td>Tusk</td>
    </tr>
    <tr>
      <th>24</th>
      <td>26</td>
      <td>3870</td>
      <td>7382.0</td>
      <td>0.524248</td>
      <td>Lion</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>1912</td>
      <td>3650.0</td>
      <td>0.523836</td>
      <td>Phantom Lancer</td>
    </tr>
    <tr>
      <th>60</th>
      <td>62</td>
      <td>3557</td>
      <td>6793.0</td>
      <td>0.523627</td>
      <td>Bounty Hunter</td>
    </tr>
    <tr>
      <th>23</th>
      <td>25</td>
      <td>4320</td>
      <td>8255.0</td>
      <td>0.523319</td>
      <td>Lina</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>4104</td>
      <td>7846.0</td>
      <td>0.523069</td>
      <td>Crystal Maiden</td>
    </tr>
    <tr>
      <th>74</th>
      <td>76</td>
      <td>818</td>
      <td>1564.0</td>
      <td>0.523018</td>
      <td>Outworld Devourer</td>
    </tr>
    <tr>
      <th>95</th>
      <td>97</td>
      <td>1794</td>
      <td>3431.0</td>
      <td>0.522880</td>
      <td>Magnus</td>
    </tr>
    <tr>
      <th>79</th>
      <td>81</td>
      <td>1225</td>
      <td>2344.0</td>
      <td>0.522611</td>
      <td>Chaos Knight</td>
    </tr>
    <tr>
      <th>66</th>
      <td>68</td>
      <td>3529</td>
      <td>6753.0</td>
      <td>0.522583</td>
      <td>Ancient Apparition</td>
    </tr>
    <tr>
      <th>90</th>
      <td>92</td>
      <td>521</td>
      <td>997.0</td>
      <td>0.522568</td>
      <td>Visage</td>
    </tr>
    <tr>
      <th>109</th>
      <td>111</td>
      <td>527</td>
      <td>1009.0</td>
      <td>0.522299</td>
      <td>Oracle</td>
    </tr>
    <tr>
      <th>58</th>
      <td>60</td>
      <td>1578</td>
      <td>3023.0</td>
      <td>0.521998</td>
      <td>Night Stalker</td>
    </tr>
    <tr>
      <th>13</th>
      <td>14</td>
      <td>4930</td>
      <td>9447.0</td>
      <td>0.521859</td>
      <td>Pudge</td>
    </tr>
    <tr>
      <th>63</th>
      <td>65</td>
      <td>550</td>
      <td>1054.0</td>
      <td>0.521822</td>
      <td>Batrider</td>
    </tr>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>4899</td>
      <td>9396.0</td>
      <td>0.521392</td>
      <td>Anti-Mage</td>
    </tr>
    <tr>
      <th>47</th>
      <td>49</td>
      <td>1001</td>
      <td>1920.0</td>
      <td>0.521354</td>
      <td>Dragon Knight</td>
    </tr>
    <tr>
      <th>86</th>
      <td>88</td>
      <td>1408</td>
      <td>2701.0</td>
      <td>0.521288</td>
      <td>Nyx Assassin</td>
    </tr>
    <tr>
      <th>41</th>
      <td>43</td>
      <td>897</td>
      <td>1721.0</td>
      <td>0.521209</td>
      <td>Death Prophet</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2395</td>
      <td>4601.0</td>
      <td>0.520539</td>
      <td>Axe</td>
    </tr>
    <tr>
      <th>20</th>
      <td>21</td>
      <td>10867</td>
      <td>20881.0</td>
      <td>0.520425</td>
      <td>Windranger</td>
    </tr>
    <tr>
      <th>39</th>
      <td>41</td>
      <td>1661</td>
      <td>3193.0</td>
      <td>0.520200</td>
      <td>Faceless Void</td>
    </tr>
    <tr>
      <th>62</th>
      <td>64</td>
      <td>1429</td>
      <td>2748.0</td>
      <td>0.520015</td>
      <td>Jakiro</td>
    </tr>
    <tr>
      <th>28</th>
      <td>30</td>
      <td>3807</td>
      <td>7321.0</td>
      <td>0.520011</td>
      <td>Witch Doctor</td>
    </tr>
    <tr>
      <th>94</th>
      <td>96</td>
      <td>965</td>
      <td>1857.0</td>
      <td>0.519655</td>
      <td>Centaur Warrunner</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>3746</td>
      <td>7210.0</td>
      <td>0.519556</td>
      <td>Mirana</td>
    </tr>
    <tr>
      <th>93</th>
      <td>95</td>
      <td>912</td>
      <td>1756.0</td>
      <td>0.519362</td>
      <td>Troll Warlord</td>
    </tr>
    <tr>
      <th>84</th>
      <td>86</td>
      <td>4246</td>
      <td>8183.0</td>
      <td>0.518881</td>
      <td>Rubick</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>5873</td>
      <td>11323.0</td>
      <td>0.518679</td>
      <td>Earthshaker</td>
    </tr>
    <tr>
      <th>67</th>
      <td>69</td>
      <td>4117</td>
      <td>7938.0</td>
      <td>0.518644</td>
      <td>Doom</td>
    </tr>
    <tr>
      <th>103</th>
      <td>105</td>
      <td>641</td>
      <td>1236.0</td>
      <td>0.518608</td>
      <td>Techies</td>
    </tr>
    <tr>
      <th>42</th>
      <td>44</td>
      <td>3775</td>
      <td>7280.0</td>
      <td>0.518544</td>
      <td>Phantom Assassin</td>
    </tr>
    <tr>
      <th>107</th>
      <td>109</td>
      <td>804</td>
      <td>1551.0</td>
      <td>0.518375</td>
      <td>Terrorblade</td>
    </tr>
    <tr>
      <th>92</th>
      <td>94</td>
      <td>1223</td>
      <td>2360.0</td>
      <td>0.518220</td>
      <td>Medusa</td>
    </tr>
    <tr>
      <th>71</th>
      <td>73</td>
      <td>5087</td>
      <td>9823.0</td>
      <td>0.517866</td>
      <td>Alchemist</td>
    </tr>
    <tr>
      <th>40</th>
      <td>42</td>
      <td>4036</td>
      <td>7794.0</td>
      <td>0.517834</td>
      <td>Wraith King</td>
    </tr>
    <tr>
      <th>55</th>
      <td>57</td>
      <td>2670</td>
      <td>5161.0</td>
      <td>0.517342</td>
      <td>Omniknight</td>
    </tr>
    <tr>
      <th>110</th>
      <td>112</td>
      <td>3981</td>
      <td>7697.0</td>
      <td>0.517214</td>
      <td>Winter Wyvern</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>1319</td>
      <td>2553.0</td>
      <td>0.516647</td>
      <td>Bane</td>
    </tr>
    <tr>
      <th>15</th>
      <td>16</td>
      <td>1627</td>
      <td>3150.0</td>
      <td>0.516508</td>
      <td>Sand King</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>8784</td>
      <td>17007.0</td>
      <td>0.516493</td>
      <td>Shadow Fiend</td>
    </tr>
    <tr>
      <th>45</th>
      <td>47</td>
      <td>1905</td>
      <td>3690.0</td>
      <td>0.516260</td>
      <td>Viper</td>
    </tr>
    <tr>
      <th>48</th>
      <td>50</td>
      <td>4338</td>
      <td>8403.0</td>
      <td>0.516244</td>
      <td>Dazzle</td>
    </tr>
    <tr>
      <th>72</th>
      <td>74</td>
      <td>6025</td>
      <td>11676.0</td>
      <td>0.516016</td>
      <td>Invoker</td>
    </tr>
    <tr>
      <th>33</th>
      <td>35</td>
      <td>1964</td>
      <td>3809.0</td>
      <td>0.515621</td>
      <td>Sniper</td>
    </tr>
    <tr>
      <th>26</th>
      <td>28</td>
      <td>5764</td>
      <td>11181.0</td>
      <td>0.515517</td>
      <td>Slardar</td>
    </tr>
    <tr>
      <th>38</th>
      <td>40</td>
      <td>1554</td>
      <td>3015.0</td>
      <td>0.515423</td>
      <td>Venomancer</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>1523</td>
      <td>2956.0</td>
      <td>0.515223</td>
      <td>Bloodseeker</td>
    </tr>
    <tr>
      <th>12</th>
      <td>13</td>
      <td>907</td>
      <td>1761.0</td>
      <td>0.515048</td>
      <td>Puck</td>
    </tr>
    <tr>
      <th>97</th>
      <td>99</td>
      <td>2145</td>
      <td>4167.0</td>
      <td>0.514759</td>
      <td>Bristleback</td>
    </tr>
    <tr>
      <th>85</th>
      <td>87</td>
      <td>2445</td>
      <td>4750.0</td>
      <td>0.514737</td>
      <td>Disruptor</td>
    </tr>
    <tr>
      <th>64</th>
      <td>66</td>
      <td>298</td>
      <td>579.0</td>
      <td>0.514680</td>
      <td>Chen</td>
    </tr>
    <tr>
      <th>49</th>
      <td>51</td>
      <td>2213</td>
      <td>4301.0</td>
      <td>0.514532</td>
      <td>Clockwerk</td>
    </tr>
    <tr>
      <th>108</th>
      <td>110</td>
      <td>1557</td>
      <td>3029.0</td>
      <td>0.514031</td>
      <td>Phoenix</td>
    </tr>
    <tr>
      <th>16</th>
      <td>17</td>
      <td>1237</td>
      <td>2407.0</td>
      <td>0.513918</td>
      <td>Storm Spirit</td>
    </tr>
    <tr>
      <th>56</th>
      <td>58</td>
      <td>522</td>
      <td>1016.0</td>
      <td>0.513780</td>
      <td>Enchantress</td>
    </tr>
    <tr>
      <th>81</th>
      <td>83</td>
      <td>894</td>
      <td>1742.0</td>
      <td>0.513203</td>
      <td>Treant Protector</td>
    </tr>
    <tr>
      <th>91</th>
      <td>93</td>
      <td>4322</td>
      <td>8426.0</td>
      <td>0.512936</td>
      <td>Slark</td>
    </tr>
    <tr>
      <th>29</th>
      <td>31</td>
      <td>2403</td>
      <td>4687.0</td>
      <td>0.512695</td>
      <td>Lich</td>
    </tr>
    <tr>
      <th>87</th>
      <td>89</td>
      <td>513</td>
      <td>1001.0</td>
      <td>0.512488</td>
      <td>Naga Siren</td>
    </tr>
    <tr>
      <th>69</th>
      <td>71</td>
      <td>3746</td>
      <td>7311.0</td>
      <td>0.512379</td>
      <td>Spirit Breaker</td>
    </tr>
    <tr>
      <th>99</th>
      <td>101</td>
      <td>1524</td>
      <td>2976.0</td>
      <td>0.512097</td>
      <td>Skywrath Mage</td>
    </tr>
    <tr>
      <th>19</th>
      <td>20</td>
      <td>2146</td>
      <td>4194.0</td>
      <td>0.511683</td>
      <td>Vengeful Spirit</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>778</td>
      <td>1524.0</td>
      <td>0.510499</td>
      <td>Morphling</td>
    </tr>
    <tr>
      <th>70</th>
      <td>72</td>
      <td>3499</td>
      <td>6856.0</td>
      <td>0.510356</td>
      <td>Gyrocopter</td>
    </tr>
    <tr>
      <th>76</th>
      <td>78</td>
      <td>475</td>
      <td>931.0</td>
      <td>0.510204</td>
      <td>Brewmaster</td>
    </tr>
    <tr>
      <th>50</th>
      <td>52</td>
      <td>625</td>
      <td>1226.0</td>
      <td>0.509788</td>
      <td>Leshrac</td>
    </tr>
    <tr>
      <th>31</th>
      <td>33</td>
      <td>1314</td>
      <td>2579.0</td>
      <td>0.509500</td>
      <td>Enigma</td>
    </tr>
    <tr>
      <th>53</th>
      <td>55</td>
      <td>2149</td>
      <td>4219.0</td>
      <td>0.509362</td>
      <td>Dark Seer</td>
    </tr>
    <tr>
      <th>105</th>
      <td>107</td>
      <td>1430</td>
      <td>2808.0</td>
      <td>0.509259</td>
      <td>Earth Spirit</td>
    </tr>
    <tr>
      <th>32</th>
      <td>34</td>
      <td>1329</td>
      <td>2610.0</td>
      <td>0.509195</td>
      <td>Tinker</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>5279</td>
      <td>10394.0</td>
      <td>0.507889</td>
      <td>Juggernaut</td>
    </tr>
    <tr>
      <th>51</th>
      <td>53</td>
      <td>1698</td>
      <td>3344.0</td>
      <td>0.507775</td>
      <td>Nature's Prophet</td>
    </tr>
    <tr>
      <th>27</th>
      <td>29</td>
      <td>1217</td>
      <td>2400.0</td>
      <td>0.507083</td>
      <td>Tidehunter</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>1322</td>
      <td>2608.0</td>
      <td>0.506902</td>
      <td>Drow Ranger</td>
    </tr>
    <tr>
      <th>43</th>
      <td>45</td>
      <td>769</td>
      <td>1522.0</td>
      <td>0.505256</td>
      <td>Pugna</td>
    </tr>
    <tr>
      <th>68</th>
      <td>70</td>
      <td>2172</td>
      <td>4302.0</td>
      <td>0.504881</td>
      <td>Ursa</td>
    </tr>
    <tr>
      <th>59</th>
      <td>61</td>
      <td>750</td>
      <td>1486.0</td>
      <td>0.504711</td>
      <td>Broodmother</td>
    </tr>
    <tr>
      <th>77</th>
      <td>79</td>
      <td>701</td>
      <td>1395.0</td>
      <td>0.502509</td>
      <td>Shadow Demon</td>
    </tr>
    <tr>
      <th>36</th>
      <td>38</td>
      <td>630</td>
      <td>1261.0</td>
      <td>0.499603</td>
      <td>Beastmaster</td>
    </tr>
    <tr>
      <th>75</th>
      <td>77</td>
      <td>491</td>
      <td>985.0</td>
      <td>0.498477</td>
      <td>Lycan</td>
    </tr>
    <tr>
      <th>80</th>
      <td>82</td>
      <td>831</td>
      <td>1670.0</td>
      <td>0.497605</td>
      <td>Meepo</td>
    </tr>
    <tr>
      <th>88</th>
      <td>90</td>
      <td>1077</td>
      <td>2167.0</td>
      <td>0.497000</td>
      <td>Keeper of the Light</td>
    </tr>
    <tr>
      <th>101</th>
      <td>103</td>
      <td>416</td>
      <td>838.0</td>
      <td>0.496420</td>
      <td>Elder Titan</td>
    </tr>
    <tr>
      <th>57</th>
      <td>59</td>
      <td>1877</td>
      <td>3782.0</td>
      <td>0.496298</td>
      <td>Huskar</td>
    </tr>
    <tr>
      <th>78</th>
      <td>80</td>
      <td>465</td>
      <td>967.0</td>
      <td>0.480869</td>
      <td>Lone Druid</td>
    </tr>
  </tbody>
</table>
</div>
</div>

</div>

</div>

</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell   ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[4]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="c1">#finding the highest winrate items</span>
<span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_won0&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">players_data</span><span class="o">.</span><span class="n">apply</span><span class="p">((</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">item_0</span> <span class="k">if</span> <span class="n">x</span><span class="o">.</span><span class="n">radiant_win</span> <span class="k">else</span> <span class="mi">0</span><span class="p">),</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>    
<span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_won1&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">players_data</span><span class="o">.</span><span class="n">apply</span><span class="p">((</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">item_1</span> <span class="k">if</span> <span class="n">x</span><span class="o">.</span><span class="n">radiant_win</span> <span class="k">else</span> <span class="mi">0</span><span class="p">),</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
<span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_won2&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">players_data</span><span class="o">.</span><span class="n">apply</span><span class="p">((</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">item_2</span> <span class="k">if</span> <span class="n">x</span><span class="o">.</span><span class="n">radiant_win</span> <span class="k">else</span> <span class="mi">0</span><span class="p">),</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
<span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_won3&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">players_data</span><span class="o">.</span><span class="n">apply</span><span class="p">((</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">item_3</span> <span class="k">if</span> <span class="n">x</span><span class="o">.</span><span class="n">radiant_win</span> <span class="k">else</span> <span class="mi">0</span><span class="p">),</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
<span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_won4&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">players_data</span><span class="o">.</span><span class="n">apply</span><span class="p">((</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">item_4</span> <span class="k">if</span> <span class="n">x</span><span class="o">.</span><span class="n">radiant_win</span> <span class="k">else</span> <span class="mi">0</span><span class="p">),</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
<span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_won5&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">players_data</span><span class="o">.</span><span class="n">apply</span><span class="p">((</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">item_5</span> <span class="k">if</span> <span class="n">x</span><span class="o">.</span><span class="n">radiant_win</span> <span class="k">else</span> <span class="mi">0</span><span class="p">),</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>

<span class="n">total_frequencies</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_0&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">value_counts</span><span class="p">()</span>
<span class="n">win_frequencies</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_won0&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">value_counts</span><span class="p">()</span>
<span class="n">item_df</span> <span class="o">=</span> <span class="n">win_frequencies</span><span class="o">.</span><span class="n">to_frame</span><span class="p">()</span><span class="o">.</span><span class="n">join</span><span class="p">(</span><span class="n">total_frequencies</span><span class="p">)</span>
<span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span><span class="mi">6</span><span class="p">):</span>
    <span class="n">total_frequencies</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_&#39;</span><span class="o">+</span><span class="nb">str</span><span class="p">(</span><span class="n">x</span><span class="p">)]</span><span class="o">.</span><span class="n">value_counts</span><span class="p">()</span>
    <span class="n">win_frequencies</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;item_won&#39;</span><span class="o">+</span><span class="nb">str</span><span class="p">(</span><span class="n">x</span><span class="p">)]</span><span class="o">.</span><span class="n">value_counts</span><span class="p">()</span>
    <span class="n">new_df</span> <span class="o">=</span> <span class="n">win_frequencies</span><span class="o">.</span><span class="n">to_frame</span><span class="p">()</span><span class="o">.</span><span class="n">join</span><span class="p">(</span><span class="n">total_frequencies</span><span class="p">)</span>
    <span class="n">item_df</span> <span class="o">=</span> <span class="n">item_df</span><span class="o">.</span><span class="n">merge</span><span class="p">(</span><span class="n">new_df</span><span class="p">,</span> <span class="n">left_index</span><span class="o">=</span><span class="kc">True</span> <span class="p">,</span><span class="n">right_index</span><span class="o">=</span><span class="kc">True</span><span class="p">)</span>

<span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_total_use&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_0&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_1&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_2&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_3&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_4&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_5&#39;</span><span class="p">]</span>
<span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_total_wins&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_won0&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_won1&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_won2&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_won3&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_won4&#39;</span><span class="p">]</span><span class="o">+</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_won5&#39;</span><span class="p">]</span>
<span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_winrate&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_total_wins&#39;</span><span class="p">]</span><span class="o">/</span><span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_total_use&#39;</span><span class="p">]</span>
<span class="n">item_df</span> <span class="o">=</span> <span class="n">item_df</span><span class="o">.</span><span class="n">drop</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
<span class="n">item_df</span> <span class="o">=</span> <span class="n">item_df</span><span class="p">[[</span><span class="s1">&#39;item_winrate&#39;</span><span class="p">,</span><span class="s1">&#39;item_total_use&#39;</span><span class="p">,</span><span class="s1">&#39;item_total_wins&#39;</span><span class="p">]]</span>
<span class="n">item_df</span> <span class="o">=</span> <span class="n">item_df</span><span class="o">.</span><span class="n">merge</span><span class="p">(</span><span class="n">item_ids</span><span class="p">,</span> <span class="n">how</span><span class="o">=</span><span class="s1">&#39;inner&#39;</span><span class="p">,</span> <span class="n">left_index</span><span class="o">=</span><span class="kc">True</span><span class="p">,</span> <span class="n">right_on</span><span class="o">=</span> <span class="s1">&#39;item_id&#39;</span><span class="p">)</span>
<span class="n">item_df</span> <span class="o">=</span> <span class="n">item_df</span><span class="o">.</span><span class="n">sort_values</span><span class="p">(</span><span class="n">by</span><span class="o">=</span><span class="s1">&#39;item_winrate&#39;</span><span class="p">,</span> <span class="n">ascending</span><span class="o">=</span><span class="kc">False</span><span class="p">)</span>
<span class="n">item_df</span>
</pre></div>

     </div>
</div>
</div>
</div>

<div class="jp-Cell-outputWrapper">
<div class="jp-Collapser jp-OutputCollapser jp-Cell-outputCollapser">
</div>


<div class="jp-OutputArea jp-Cell-outputArea">

<div class="jp-OutputArea-child">

    
    <div class="jp-OutputPrompt jp-OutputArea-prompt">Out[4]:</div>



<div class="jp-RenderedHTMLCommon jp-RenderedHTML jp-OutputArea-output jp-OutputArea-executeResult" data-mime-type="text/html">
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>item_winrate</th>
      <th>item_total_use</th>
      <th>item_total_wins</th>
      <th>item_id</th>
      <th>item_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>149</th>
      <td>0.593640</td>
      <td>283</td>
      <td>168</td>
      <td>241</td>
      <td>tango_single</td>
    </tr>
    <tr>
      <th>36</th>
      <td>0.563462</td>
      <td>1560</td>
      <td>879</td>
      <td>38</td>
      <td>clarity</td>
    </tr>
    <tr>
      <th>136</th>
      <td>0.562682</td>
      <td>2058</td>
      <td>1158</td>
      <td>215</td>
      <td>shadow_amulet</td>
    </tr>
    <tr>
      <th>56</th>
      <td>0.562448</td>
      <td>16926</td>
      <td>9520</td>
      <td>60</td>
      <td>point_booster</td>
    </tr>
    <tr>
      <th>50</th>
      <td>0.556808</td>
      <td>1153</td>
      <td>642</td>
      <td>54</td>
      <td>relic</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>105</th>
      <td>0.478712</td>
      <td>6741</td>
      <td>3227</td>
      <td>156</td>
      <td>satanic</td>
    </tr>
    <tr>
      <th>81</th>
      <td>0.478436</td>
      <td>11153</td>
      <td>5336</td>
      <td>110</td>
      <td>refresher</td>
    </tr>
    <tr>
      <th>139</th>
      <td>0.464953</td>
      <td>4551</td>
      <td>2116</td>
      <td>220</td>
      <td>travel_boots_2</td>
    </tr>
    <tr>
      <th>43</th>
      <td>0.438914</td>
      <td>221</td>
      <td>97</td>
      <td>45</td>
      <td>courier</td>
    </tr>
    <tr>
      <th>68</th>
      <td>0.421053</td>
      <td>38</td>
      <td>16</td>
      <td>84</td>
      <td>flying_courier</td>
    </tr>
  </tbody>
</table>
<p>149 rows × 5 columns</p>
</div>
</div>

</div>

</div>

</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell   ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[5]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">random</span><span class="o">.</span><span class="n">seed</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
<span class="n">np</span><span class="o">.</span><span class="n">random</span><span class="o">.</span><span class="n">seed</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
<span class="n">n</span> <span class="o">=</span> <span class="mi">143</span>


<span class="n">labels</span> <span class="o">=</span> <span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_name&#39;</span><span class="p">]</span>
<span class="n">x</span> <span class="o">=</span> <span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_id&#39;</span><span class="p">]</span>
<span class="n">y</span> <span class="o">=</span> <span class="n">item_df</span><span class="p">[</span><span class="s1">&#39;item_winrate&#39;</span><span class="p">]</span>


<span class="n">fig</span><span class="p">,</span> <span class="n">ax</span> <span class="o">=</span> <span class="n">pl</span><span class="o">.</span><span class="n">subplots</span><span class="p">(</span><span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">20</span><span class="p">,</span> <span class="mi">10</span><span class="p">));</span>

<span class="n">linear_regressor</span> <span class="o">=</span> <span class="n">LinearRegression</span><span class="p">()</span>
<span class="n">linear_regressor</span><span class="o">.</span><span class="n">fit</span><span class="p">(</span><span class="n">x</span><span class="o">.</span><span class="n">values</span><span class="o">.</span><span class="n">reshape</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span><span class="mi">1</span><span class="p">),</span> <span class="n">y</span><span class="p">)</span>
<span class="n">x_pred</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">linspace</span><span class="p">(</span><span class="n">x</span><span class="o">.</span><span class="n">min</span><span class="p">(),</span> <span class="n">x</span><span class="o">.</span><span class="n">max</span><span class="p">(),</span> <span class="n">num</span><span class="o">=</span><span class="mi">200</span><span class="p">)</span><span class="o">.</span><span class="n">reshape</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
<span class="n">y_pred</span> <span class="o">=</span> <span class="n">linear_regressor</span><span class="o">.</span><span class="n">predict</span><span class="p">(</span><span class="n">x_pred</span><span class="p">)</span>  
<span class="n">ax</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">x_pred</span><span class="p">,</span> <span class="n">y_pred</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="s2">&quot;#FFCCCC&quot;</span><span class="p">,</span> <span class="n">lw</span><span class="o">=</span><span class="mi">4</span><span class="p">)</span>

<span class="n">ax</span><span class="o">.</span><span class="n">scatter</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s1">&#39;Item ID&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylabel</span><span class="p">(</span><span class="s1">&#39;Winrate&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">title</span><span class="p">(</span><span class="s2">&quot;Item ID vs Winrate&quot;</span><span class="p">)</span>



<span class="n">ann</span> <span class="o">=</span> <span class="p">[]</span>
<span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="n">n</span><span class="p">):</span>
    <span class="n">ann</span><span class="o">.</span><span class="n">append</span><span class="p">(</span><span class="n">ax</span><span class="o">.</span><span class="n">annotate</span><span class="p">(</span><span class="n">labels</span><span class="p">[</span><span class="n">i</span><span class="p">],</span> <span class="n">xy</span> <span class="o">=</span> <span class="p">(</span><span class="n">x</span><span class="p">[</span><span class="n">i</span><span class="p">]</span> <span class="o">+</span> <span class="mi">1</span><span class="p">,</span> <span class="n">y</span><span class="p">[</span><span class="n">i</span><span class="p">]</span><span class="o">-</span><span class="mf">.0005</span><span class="p">)))</span>

<span class="n">mask</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="n">fig</span><span class="o">.</span><span class="n">canvas</span><span class="o">.</span><span class="n">get_width_height</span><span class="p">(),</span> <span class="nb">bool</span><span class="p">)</span>

<span class="n">fig</span><span class="o">.</span><span class="n">canvas</span><span class="o">.</span><span class="n">draw</span><span class="p">()</span>

<span class="k">for</span> <span class="n">a</span> <span class="ow">in</span> <span class="n">ann</span><span class="p">:</span>
    <span class="n">bbox</span> <span class="o">=</span> <span class="n">a</span><span class="o">.</span><span class="n">get_window_extent</span><span class="p">()</span>
    <span class="n">x0</span> <span class="o">=</span> <span class="nb">int</span><span class="p">(</span><span class="n">bbox</span><span class="o">.</span><span class="n">x0</span><span class="p">)</span>
    <span class="n">x1</span> <span class="o">=</span> <span class="nb">int</span><span class="p">(</span><span class="n">math</span><span class="o">.</span><span class="n">ceil</span><span class="p">(</span><span class="n">bbox</span><span class="o">.</span><span class="n">x1</span><span class="p">))</span>
    <span class="n">y0</span> <span class="o">=</span> <span class="nb">int</span><span class="p">(</span><span class="n">bbox</span><span class="o">.</span><span class="n">y0</span><span class="p">)</span>
    <span class="n">y1</span> <span class="o">=</span> <span class="nb">int</span><span class="p">(</span><span class="n">math</span><span class="o">.</span><span class="n">ceil</span><span class="p">(</span><span class="n">bbox</span><span class="o">.</span><span class="n">y1</span><span class="p">))</span>

    <span class="n">s</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">s_</span><span class="p">[</span><span class="n">x0</span><span class="p">:</span><span class="n">x1</span><span class="o">+</span><span class="mi">1</span><span class="p">,</span> <span class="n">y0</span><span class="p">:</span><span class="n">y1</span><span class="o">+</span><span class="mi">1</span><span class="p">]</span>
    <span class="k">if</span> <span class="n">np</span><span class="o">.</span><span class="n">any</span><span class="p">(</span><span class="n">mask</span><span class="p">[</span><span class="n">s</span><span class="p">]):</span>
        <span class="n">a</span><span class="o">.</span><span class="n">set_visible</span><span class="p">(</span><span class="kc">False</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">mask</span><span class="p">[</span><span class="n">s</span><span class="p">]</span> <span class="o">=</span> <span class="kc">True</span>
</pre></div>

     </div>
</div>
</div>
</div>

<div class="jp-Cell-outputWrapper">
<div class="jp-Collapser jp-OutputCollapser jp-Cell-outputCollapser">
</div>


<div class="jp-OutputArea jp-Cell-outputArea">

<div class="jp-OutputArea-child">

    
    <div class="jp-OutputPrompt jp-OutputArea-prompt"></div>




<div class="jp-RenderedImage jp-OutputArea-output ">
<img src="data:image/png;base64,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RKRQKeAkIiIiIiIiIiKFSgEnEREREREREREpVAo4iYiIiIiIiIhIoVLASURERERERERECpUCTiIiIiIiIiIiUqgUcBIRERERERERkUKlgJOIiIiIiIiIiBQqBZxERERERERERKRQKeAkIiIiIiIiIiKFSgEnEREREREREREpVAo4iYiIiIiIiIhIoVLASURERERERERECpUCTiIiIiIiIiIiUqgUcBIRERERERERkUKlgJOIiIiIiIiIiBQqBZxERERERERERKRQKeAkIiIiIiIiIiKFSgEnEREREREREREpVAo4iYiIiIiIiIhIoVLASURERERERERECpUCTiIiIiIiIiIiUqgUcBIRERERERERkUKlgJOIiIiIiIiIiBQqBZxERERERERERKRQKeAkIiIiIiIiIiKFSgEnEREREREREREpVAo4iYiIiIiIiIhIoVLASURERERERERECpUCTiIiIiIiIiIiUqgUcBIRERERERERkUKlgJOIiIiIiIiIiBQqBZxERERERERERKRQKeAkIiIiIiIiIiKFSgEnEREREREREREpVAo4iYiIiIiIiIhIoSrSgJNhGI8bhnHIMIxfDcMYZWO7v2EY8YZhRKb/G5deXi9TWaRhGP8YhvFq+rbxhmHEZtr2ZFGeg4iIiIiIiIiIXJtiRVWxYRj2wHTgMeAPINQwjJWmaR7ItutW0zTbZy4wTfMQYMlUTyywPNMuH5umGVxUfRcRERERERERketXlCOcGgO/mqb5m2maV4HFwNPXUU9r4Khpmr8Xau9ERERERERERKRIFGXAqRZwItPzP9LLsvMxDGOvYRg/GobhZGN7d2BRtrKXDcOIMgxjrmEYlWw1bhjGAMMwwgzDCDtz5sx1nYCIiIiIiIiIiFy7ogw4GTbKzGzP9wC1TdN0A6YBK7JUYBjFgQ7At5mKZwAPkTbl7hTwoa3GTdOcZZqmp2mantWqVbue/ouIiIiIiIiIyHUoyoDTH8CDmZ4/AJzMvINpmv+Ypnkx/fEPgINhGFUz7fIEsMc0zdOZjjltmmaKaZqpwGzSpu6JiIiIiIiIiMhtoigDTqHAI4ZhOKaPVOoOrMy8g2EY9xuGYaQ/bpzen3OZdulBtul0hmHUyPS0ExBdBH0XEREREREREZHrVGSr1JmmmWwYxsvAGsAemGua5n7DMAamb58JdAEGGYaRDCQA3U3TNAEMwyhN2gp3L2Wr+gPDMCykTc+LsbFdRERERERERERuISM9vnNX8/T0NMPCwm51N0RERERERERE7hqGYYSbpulpa1tRTqkTEREREREREZF7kAJOIiIiIiIiIiJSqBRwEhERERERERGRQqWAk4iIiIiIiIiIFCoFnEREREREREREpFAp4CQiIiIiIiIiIoVKAScRERERERERESlUCjiJiIiIiIiIiEihUsBJREREREREREQKlQJOIiIiIiIiIiJSqBRwEhERERERERGRQqWAk4iIiIiIiIiIFCoFnEREREREREREpFAp4CQiIiIiIiIiIoVKAScRERERERERESlUCjiJiIiIiIiIiEihUsBJREREREREREQKlQJOIiIiIiIiIiJSqBRwEhERERERERGRQqWAk4iIiIiIiIiIFCoFnEREREREREREpFAp4CQiIiIiIiIiIoVKAScRydf48eMJDg6+pmNWrlzJpEmTAFixYgUHDhwoiq6JiIiIiIjIbUgBJxEpdMnJyXTo0IFRo0YBCjiJiIiIiIjca4rd6g6IyM23IiKWyWsOcTIugZoVSxEUUI+O7rWs2xcsWEBwcDCGYeDq6spDDz1k3TZ79mxmzZrF1atXefjhh/nPf/5D6dKlCQwMpHLlykRERODh4YGLiwthYWH07NmTlStXsnnzZt555x2WLl1K165d2bNnDwBHjhyhe/fuhIeH3/TXQURERERERIqGRjiJ3GNWRMQyetk+YuMSMIHYuARGL9vHiohYAPbv38/EiRPZsGEDe/fu5ZNPPslyfOfOnQkNDWXv3r00aNCAOXPmWLcdPnyYdevW8eGHH1rLmjZtSocOHZg8eTKRkZE89NBDVKhQgcjISADmzZtHYGBgUZ+2iIiIiMgNqVOnDmfPnr3u4/39/QkLCyvEHhWtwMBAlixZkuc+8+fP5+TJkzepR3KnUcBJ5B4zec0hEpJSspQlJKUwec0hADZs2ECXLl2oWrUqAJUrV86yb3R0NM2bN8fFxYWFCxeyf/9+67auXbtib2+fbx9efPFF5s2bR0pKCiEhIfTs2fNGT+uu9eKLL+Y7HbEgUxaL8gInJiaGr7/+ukjqFhEREZHblwJOkhcFnETuMSfjEvIsN00TwzByPT4wMJBPP/2Uffv28eabb5KYmGjdVqZMmQL14ZlnnuHHH3/k+++/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lfL19b3k70CdOnX0+uuve/wAf+rUKY0aNUqbNm2Sr6+vzpw5U2j966+/rri4OMXGxqpq1apl+l6WRHHv9z0t65Q44Fu0aJHi4+M1bdo0BQYG6vDhw/Ly8tKvv/6qVq1a6fDhw1q0aJHefPNNXbhwQf/+7/+uxYsXq3r16vrggw/0wgsvyNvbW7Vr1y404WpBv/76q6Kjo7V//361adNGqampmjt3rkJCQlSzZk33B43ly5dr9erVWrRokT7++GO99NJLunDhgm699VbFxMSoYcOGmjp1qo4fP67U1FTVr19fc+bM0dChQ3Xq1CmFhoaWeP4Pp23cuFE1a9YsFDj169evVIFTVlaWEhMTlZmZ6X7a6Lx588qiugAcVtwcRRs3btTatWu1detWVa9eXT169HDPOVRQ/rxAgwYN0gsvvKBevXqpffv27gBo27ZtWrdunZYsWaLXX39d69ev1/z58/XNN9/ok08+kcvlUlJSkubMmaOGDRtq586dysnJKVVvnMvNVRQTE6NTp04pISFBPj4+at68uftaPM1rlJOTozp16vAEZVx3mMMJqOAu9zSK9evXKyoqyj3soV69epJyx7h7eXmpbdu2+uGHHyRJcXFxiouLU1BQkIKDg7V//34dOHBA69atU0JCgjp06CCXy6V169bp8OHDHp9Qkf8Ny+DBg+VyuTRmzBidOHHiGr4jQOVWXJv39/fX2rVrNWnSJH355ZeqXbu2JLm/jW3fvr1SU1Ml5bb36dOny+Vyuf+n/MiRI4qLi9M777wjl8uljh076vTp0zpw4IAkKTQ0VL/73e/k7e2toUOHavPmzSV+Uk2+r7/+Wt26dZOvr6+kf/1NknLndvjss8+0YsWKChE2ScW/359//rkaN26snTt3avfu3erdu7ckuQO+cePGXTIvVe3atRUYGOh+DPTHH3+siIgI+fj4aODAgdq+fbt27typNm3a6K233pL0r6EcO3fu1EcffVRkXd944w3VrVtXu3bt0vPPP6+EhITLXl/Xrl319ddfKzExUffff79eeeUV97qEhAStWrVK7733nl544QV17dpViYmJioyM1JEjR0r2BpZAVlaWRowYoYCAAEVFRenXX38tNPdYfHy8evToodTUVM2fP1+vvvqqXC6XvvjiC3300UeaOHGiXC6XDh06pEOHDql3795q37697r77bvcTjqKjo/X000+rZ8+eGjVqlB544AH3UJpDhw656/LWW2/pqaeeci///e9/19NPP+3YtQK4OhfPRVSw/Oeff1bdunVVvXp17d+/392rJicnxz136XvvvaeuXbtKyh22FhERoXHjxmnkyJGScnsA/fzzz7r33ns1a9Ysd4Bz6NAhdezYUX/5y19Uv359HT16VD///LMaNWokLy8vLV68WNnZ2ZdWrAj/+Mc/dObMGWVkZCg2NlZdunQptP7nn3/WbbfdJh8fH23YsEHff/+9pNy5lz788ENlZGQoPT1dH3/8sSTplltuka+vrz744ANJuT2kdu7cWeL6OGHChAlyuVyFfoobPgdI9HACKrzLPY3CWuvxm5CCH+byv6m21mry5MkaM2ZMoW3nzJmjESNGeByfffETKmbNmsU3LEAZKq7Nt2zZUgkJCfr00081efJkd8+k/Pbu7e2trKwsSbntfcWKFZdMyGmt1Zw5cxQREVGofOPGjR6/kS3tk2qK+pskSX5+fkpKStKxY8fcgVR5K+799vf31x/+8AdNmjRJ/fr109133y2pcMC3cuXKS/YdMmSIli5dqp49e2rJkiUaP368JGn37t167rnndPbsWZ07d87938DTUA5PNm/erCeeeEJS7nsZEBBw2es7duyYhgwZohMnTujChQuF3vfIyEj3kMZNmza5r6Vv376qW7fuZY+d73JDElNSUvTWW2+pS5cuevjhh91Dti/WvHlzjR07VjVr1nRP5BsZGal+/fq5h4WHhYUV2cP222+/1dq1a+Xt7a2NGzdq5syZWr16daFz3H///QoICNArr7wiHx8fLVy4UP/zP/9T4msFULYmRrQq1OtUknuOot5t62v+/PkKCAhQq1at1KlTJ0m5T0Lbs2eP2rdvr9q1a2vp0qXufYcPH66VK1e675fp6em677779Ntvv8laq1dffTX3vBMn6sCBA7LWKiwsTIGBgRo/frwGDRqkDz74QD179rzkiWvF6dq1qx588EEdPHhQw4YNu6Q38PDhw/Wf//mfCgkJkcvlck9NERwcrCFDhsjlcqlZs2bu+46U2ytq3Lhxeumll5SZman7779fgYGBpXyHr9zcuXOv2blQeRA4ARVc4zrVlObhA1H+N0BhYWEaMGCAnnrqKd16662XDF8pKCIiQs8//7yGDx+umjVrKi0tTT4+PgoLC9N9992np556SrfddpvOnDmj9PR01ahRQzfddJMGDRqkO++8U9HR0YW+YRk8eLCstdq1a9c1veEBlVlxbf748eOqV6+eHnjgAdWsWbPY+RAiIiI0Z84czZkzR8YYJSYmKigoSBEREZo3b5569eolHx8fffvtt2rSJDcc2LZtm7777js1a9ZMS5cu1ejRo/Xjjz9e8ndA0iVPusnXuXNnTZgwQd999517SF1+L6egoCCNGzdOkZGRWrNmjXvC8fJU3PtdmoCvoMjISE2ePFlnzpxRQkKCevXqJSm3F05sbKwCAwO1aNEi9/wWnoZy5A/9KKi4YW4FQ76CQ0wee+wxPf3004qMjNTGjRs1depU97rLPa66JC437FuSmjZt6v52/4EHHtDs2bNLfR5JhXrY5jt//rz79eDBg+Xt7V3sMWrUqKFevXpp9erVatOmjTIzM+Xv739F9QHgvPy/G0WF2J999tkl++QPJ37xxRcvWbd582Y9/PDD7r8NjRo1cj+JrSBPXx60aNFCu3btci/nfzHbvHnzYueli46Odt8ri6pr/fr1tXXrVo/bPPvss3r22WcvKff19dXnn39e5HmBiojACajgivumR5LatWunZ599Vt27d5e3t7eCgoKKPFZ4eLj27dunzp07S5Jq1qypd999V23bttVLL72k8PBw5eTkyMfHR3PnzlW1atU8PqGivL9hASqz4tp8cnKyJk6cKC8vL/n4+GjevHnunh8Xe/755/Xkk08qICBA1lo1b95cq1ev1iOPPKLU1FQFBwfLWqsGDRooNjZWUm5Y9Mwzzyg5OVndunXTgAEDlJyc7PHvQHR0tMaOHeueNDxfgwYN9Oabb2rgwIHKyclxP+UnX9euXTVz5kz17dtX//jHP8r9KWjFvd+lCfgKqlmzpkJDQ/XEE0+oX79+7g866enpatSokTIzMxUTE+MO+vKHcnTs2FEff/yxjh496jFw6tq1q5YtW6aePXtq7969Sk5Odq9r2LCh9u3bp1atWunDDz9UrVq1JOUO28g/z9tvv11knbt166aYmBg999xz+uyzz/TTTz+V6FovN8Gv5HkukypVqrh/pzzNweLJ5eYwKWnvg0ceeUR//etf1bp1a/cwGwAVR/+gJqV+cIMnAwYM0KFDh5hnFChHBE5ABXe5b3okacSIERoxYkSRx8j/NkWSnnjiCfeQjIKG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GR63k6QePXpo0qRJCg0NVcuWLfXll19KKhymnjt3TiNHjnTPg7NixYpyew9wdSZGtFI1H+9CZdV8vDUxolWR+9xyyy3y9fXVBx98ICn3b+vOnTslSWFhYe7haNnZ2frll18u2b9bt25aunSpsrOzderUKW3atEmhoaFFnq9169ZKTU3VwYMHJUmLFy9W9+7dS3ehecLDwzV//nz339ozZ844dvyDBw+629GOHTt04cIF3XrrrVdUTwCeETgBAIAK4+LJal988UV16NBBAQEBmjJliiRp7NixOnz4sCIjI/Xqq69q6tSpGj16tMLDw/XQQw/p1KlTGjRokDp06KAOHTpoy5YtkqQvvvhCLpdLLpdLQUFBSk9Pl5T7YdzTpLcJCQnq3r272rdvr4iICPcQlB49euhPf/qTunfvrtdee60c3iVUZLGJaeoyfb18n/lEXaavd0/mLEmff/65GjdurJ07d2r37t3q3bu3Hn30UW3fvl27d+9WRkaGVq9eraioKIWEhCgmJkZJSUmqVq2ax+3yZWVladu2bZo1a5ZeeOGFS+r04osvqnbt2kpOTtauXbvUq1eva/JewHn9g5ro5YH+alKnmoykJnWq6eWB/pd9Sl1MTIzeeustBQYGql27dlq1apUk6bXXXtOGDRvk7++v9u3ba8+ePZfsO2DAAAUEBCgwMFC9evXSK6+8on/7t38r8lw333yzFi5cqMGDB8vf319eXl4aO3bsFV3vI488ojvuuMN9/vfee8+x469YsUJ+fn5yuVyaMGGCli5dysThgNPyH3VZmX/at29vAQBAxfDhjmP2rpfX2eaTVtu7Xl5nP9xxzL3uu+++s8YYu3XrVrtmzRo7atQom5OTY7Ozs23fvn3tF198Ya21tlmzZvbUqVPWWmunTJlig4OD7a+//mqttXbo0KH2yy+/tNZa+/3339vWrVtba63t16+f3bx5s7XW2vT0dJuZmWk3bNhgb7nlFnv06FGbnZ1tO3XqZL/88kt74cIF27lzZ3vy5ElrrbVLliyxI0eOtNZa2717dztu3Lhr8E7hevPhjmO29XOf2WaTVrt/Wj/3mft3PCUlxTZv3tz+8Y9/tJs2bbLWWrt8+XIbGhpq/fz8bOPGje3LL79src39Pdu+fbv72MVtl/97/X//93/2zjvvtNZau2HDBtu3b19rrbXBwcH222+/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
"
>
</div>

</div>

</div>

</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell   ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[13]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="c1">#Finding out if K/D/A predicts win%</span>
<span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;radiant_win&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[</span><span class="s1">&#39;radiant_win&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">apply</span><span class="p">((</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="mi">1</span> <span class="k">if</span> <span class="n">x</span> <span class="k">else</span> <span class="mi">0</span><span class="p">))</span>
<span class="n">mdata</span> <span class="o">=</span> <span class="n">players_data</span><span class="p">[[</span><span class="s1">&#39;kills&#39;</span><span class="p">,</span><span class="s1">&#39;deaths&#39;</span><span class="p">,</span><span class="s1">&#39;assists&#39;</span><span class="p">,</span><span class="s1">&#39;radiant_win&#39;</span><span class="p">]]</span>

<span class="n">mod</span> <span class="o">=</span> <span class="n">smf</span><span class="o">.</span><span class="n">ols</span><span class="p">(</span><span class="n">formula</span><span class="o">=</span><span class="s1">&#39;radiant_win ~ kills + deaths + assists&#39;</span><span class="p">,</span> <span class="n">data</span><span class="o">=</span><span class="n">mdata</span><span class="p">)</span>
<span class="n">res</span> <span class="o">=</span> <span class="n">mod</span><span class="o">.</span><span class="n">fit</span><span class="p">()</span>
<span class="nb">print</span><span class="p">(</span><span class="n">res</span><span class="o">.</span><span class="n">summary</span><span class="p">())</span>
</pre></div>

     </div>
</div>
</div>
</div>

<div class="jp-Cell-outputWrapper">
<div class="jp-Collapser jp-OutputCollapser jp-Cell-outputCollapser">
</div>


<div class="jp-OutputArea jp-Cell-outputArea">

<div class="jp-OutputArea-child">

    
    <div class="jp-OutputPrompt jp-OutputArea-prompt"></div>


<div class="jp-RenderedText jp-OutputArea-output" data-mime-type="text/plain">
<pre>                            OLS Regression Results                            
==============================================================================
Dep. Variable:            radiant_win   R-squared:                       0.003
Model:                            OLS   Adj. R-squared:                  0.003
Method:                 Least Squares   F-statistic:                     469.4
Date:                Thu, 15 Dec 2022   Prob (F-statistic):          1.27e-304
Time:                        23:30:57   Log-Likelihood:            -3.6181e+05
No. Observations:              499963   AIC:                         7.236e+05
Df Residuals:                  499959   BIC:                         7.237e+05
Df Model:                           3                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P&gt;|t|      [0.025      0.975]
------------------------------------------------------------------------------
Intercept      0.5936      0.002    280.340      0.000       0.589       0.598
kills         -0.0019      0.000    -13.788      0.000      -0.002      -0.002
deaths        -0.0046      0.000    -24.350      0.000      -0.005      -0.004
assists       -0.0022      0.000    -18.980      0.000      -0.002      -0.002
==============================================================================
Omnibus:                  1719466.954   Durbin-Watson:                   2.009
Prob(Omnibus):                  0.000   Jarque-Bera (JB):            82395.535
Skew:                          -0.075   Prob(JB):                         0.00
Kurtosis:                       1.017   Cond. No.                         51.5
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
</pre>
</div>
</div>

</div>

</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell jp-mod-noOutputs  ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[&nbsp;]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">profanities_text</span> <span class="o">=</span> <span class="nb">open</span><span class="p">(</span><span class="s1">&#39;bad_words.txt&#39;</span><span class="p">,</span> <span class="s1">&#39;r&#39;</span><span class="p">)</span>
<span class="n">profanities</span> <span class="o">=</span> <span class="n">profanities_text</span><span class="o">.</span><span class="n">read</span><span class="p">()</span><span class="o">.</span><span class="n">split</span><span class="p">(</span><span class="s1">&#39;,&#39;</span><span class="p">)</span>
</pre></div>

     </div>
</div>
</div>
</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell   ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[8]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="k">def</span> <span class="nf">slur_count</span><span class="p">(</span><span class="n">text</span><span class="p">):</span>
    <span class="nb">sum</span> <span class="o">=</span> <span class="mi">0</span>
    <span class="k">for</span> <span class="n">phrase</span> <span class="ow">in</span> <span class="n">profanities</span><span class="p">:</span>
        <span class="nb">sum</span> <span class="o">+=</span> <span class="nb">str</span><span class="p">(</span><span class="n">text</span><span class="p">)</span><span class="o">.</span><span class="n">lower</span><span class="p">()</span><span class="o">.</span><span class="n">count</span><span class="p">(</span><span class="n">phrase</span><span class="p">)</span>
    <span class="k">return</span> <span class="nb">sum</span>



<span class="n">chat</span> <span class="o">=</span> <span class="n">chat</span><span class="o">.</span><span class="n">merge</span><span class="p">(</span><span class="n">players_data</span><span class="p">[[</span><span class="s1">&#39;match_id&#39;</span><span class="p">,</span><span class="s1">&#39;hero_id&#39;</span><span class="p">,</span><span class="s1">&#39;player_slot&#39;</span><span class="p">,</span><span class="s1">&#39;localized_name&#39;</span><span class="p">]],</span> <span class="n">how</span><span class="o">=</span><span class="s1">&#39;inner&#39;</span><span class="p">,</span> <span class="n">left_on</span><span class="o">=</span><span class="p">[</span><span class="s1">&#39;match_id&#39;</span><span class="p">,</span><span class="s1">&#39;slot&#39;</span><span class="p">],</span> <span class="n">right_on</span><span class="o">=</span><span class="p">[</span><span class="s1">&#39;match_id&#39;</span><span class="p">,</span><span class="s1">&#39;player_slot&#39;</span><span class="p">])</span>

<span class="n">chat</span><span class="p">[</span><span class="s1">&#39;detected&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">chat</span><span class="p">[</span><span class="s1">&#39;key&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">apply</span><span class="p">(</span><span class="n">slur_count</span><span class="p">)</span>

<span class="n">swears_per_hero</span> <span class="o">=</span> <span class="n">hero_names</span>


<span class="n">hero_list</span> <span class="o">=</span> <span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">*</span> <span class="mi">120</span>
<span class="k">for</span> <span class="n">index</span><span class="p">,</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">chat</span><span class="o">.</span><span class="n">iterrows</span><span class="p">():</span>
    <span class="n">hero_list</span><span class="p">[</span><span class="n">row</span><span class="o">.</span><span class="n">hero_id</span><span class="p">]</span> <span class="o">+=</span> <span class="n">row</span><span class="o">.</span><span class="n">detected</span>

<span class="n">hero_swears</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">Series</span><span class="p">(</span><span class="n">hero_list</span><span class="p">)</span><span class="o">.</span><span class="n">to_frame</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="s1">&#39;profanities&#39;</span><span class="p">)</span>
<span class="n">swears_per_hero</span> <span class="o">=</span> <span class="n">swears_per_hero</span><span class="o">.</span><span class="n">merge</span><span class="p">(</span><span class="n">hero_swears</span><span class="p">,</span> <span class="n">left_on</span><span class="o">=</span><span class="s1">&#39;hero_id&#39;</span><span class="p">,</span><span class="n">right_index</span><span class="o">=</span><span class="kc">True</span><span class="p">)</span>
<span class="c1">#lost after here for some reason</span>
<span class="n">swears_per_hero</span> <span class="o">=</span> <span class="n">hero_play_df</span><span class="o">.</span><span class="n">merge</span><span class="p">(</span><span class="n">swears_per_hero</span><span class="p">,</span> <span class="n">on</span><span class="o">=</span><span class="s1">&#39;hero_id&#39;</span><span class="p">)</span>
<span class="n">swears_per_hero</span> <span class="o">=</span> <span class="n">swears_per_hero</span><span class="o">.</span><span class="n">drop</span><span class="p">(</span><span class="n">labels</span><span class="o">=</span><span class="p">[</span><span class="s1">&#39;name&#39;</span><span class="p">,</span><span class="s1">&#39;localized_name_y&#39;</span><span class="p">],</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span><span class="o">.</span><span class="n">rename</span><span class="p">(</span><span class="n">columns</span><span class="o">=</span><span class="p">{</span><span class="s1">&#39;localized_name_x&#39;</span><span class="p">:</span><span class="s1">&#39;localized_name&#39;</span><span class="p">})</span>
<span class="n">swears_per_hero</span><span class="p">[</span><span class="s1">&#39;profanities_per_play&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">swears_per_hero</span><span class="p">[</span><span class="s1">&#39;profanities&#39;</span><span class="p">]</span><span class="o">/</span><span class="n">swears_per_hero</span><span class="p">[</span><span class="s1">&#39;hero_plays&#39;</span><span class="p">]</span>
<span class="n">swears_per_hero</span> <span class="o">=</span> <span class="n">swears_per_hero</span><span class="o">.</span><span class="n">sort_values</span><span class="p">(</span><span class="n">by</span><span class="o">=</span><span class="s1">&#39;profanities_per_play&#39;</span><span class="p">,</span> <span class="n">ascending</span><span class="o">=</span><span class="kc">False</span><span class="p">)</span>
<span class="n">swears_per_hero</span>
</pre></div>

     </div>
</div>
</div>
</div>

<div class="jp-Cell-outputWrapper">
<div class="jp-Collapser jp-OutputCollapser jp-Cell-outputCollapser">
</div>


<div class="jp-OutputArea jp-Cell-outputArea">

<div class="jp-OutputArea-child">

    
    <div class="jp-OutputPrompt jp-OutputArea-prompt">Out[8]:</div>



<div class="jp-RenderedHTMLCommon jp-RenderedHTML jp-OutputArea-output jp-OutputArea-executeResult" data-mime-type="text/html">
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>hero_id</th>
      <th>hero_won</th>
      <th>hero_plays</th>
      <th>hero_winrate</th>
      <th>localized_name</th>
      <th>profanities</th>
      <th>profanities_per_play</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>105</th>
      <td>82</td>
      <td>831</td>
      <td>1670.0</td>
      <td>0.497605</td>
      <td>Meepo</td>
      <td>379</td>
      <td>0.226946</td>
    </tr>
    <tr>
      <th>38</th>
      <td>65</td>
      <td>550</td>
      <td>1054.0</td>
      <td>0.521822</td>
      <td>Batrider</td>
      <td>209</td>
      <td>0.198292</td>
    </tr>
    <tr>
      <th>54</th>
      <td>105</td>
      <td>641</td>
      <td>1236.0</td>
      <td>0.518608</td>
      <td>Techies</td>
      <td>240</td>
      <td>0.194175</td>
    </tr>
    <tr>
      <th>93</th>
      <td>107</td>
      <td>1430</td>
      <td>2808.0</td>
      <td>0.509259</td>
      <td>Earth Spirit</td>
      <td>534</td>
      <td>0.190171</td>
    </tr>
    <tr>
      <th>100</th>
      <td>70</td>
      <td>2172</td>
      <td>4302.0</td>
      <td>0.504881</td>
      <td>Ursa</td>
      <td>813</td>
      <td>0.188982</td>
    </tr>
    <tr>
      <th>50</th>
      <td>95</td>
      <td>912</td>
      <td>1756.0</td>
      <td>0.519362</td>
      <td>Troll Warlord</td>
      <td>330</td>
      <td>0.187927</td>
    </tr>
    <tr>
      <th>8</th>
      <td>98</td>
      <td>1548</td>
      <td>2934.0</td>
      <td>0.527607</td>
      <td>Timbersaw</td>
      <td>546</td>
      <td>0.186094</td>
    </tr>
    <tr>
      <th>87</th>
      <td>10</td>
      <td>778</td>
      <td>1524.0</td>
      <td>0.510499</td>
      <td>Morphling</td>
      <td>283</td>
      <td>0.185696</td>
    </tr>
    <tr>
      <th>37</th>
      <td>14</td>
      <td>4930</td>
      <td>9447.0</td>
      <td>0.521859</td>
      <td>Pudge</td>
      <td>1746</td>
      <td>0.184821</td>
    </tr>
    <tr>
      <th>101</th>
      <td>61</td>
      <td>750</td>
      <td>1486.0</td>
      <td>0.504711</td>
      <td>Broodmother</td>
      <td>257</td>
      <td>0.172948</td>
    </tr>
    <tr>
      <th>19</th>
      <td>22</td>
      <td>2412</td>
      <td>4589.0</td>
      <td>0.525605</td>
      <td>Zeus</td>
      <td>785</td>
      <td>0.171061</td>
    </tr>
    <tr>
      <th>11</th>
      <td>106</td>
      <td>3973</td>
      <td>7533.0</td>
      <td>0.527413</td>
      <td>Ember Spirit</td>
      <td>1273</td>
      <td>0.168990</td>
    </tr>
    <tr>
      <th>83</th>
      <td>89</td>
      <td>513</td>
      <td>1001.0</td>
      <td>0.512488</td>
      <td>Naga Siren</td>
      <td>168</td>
      <td>0.167832</td>
    </tr>
    <tr>
      <th>89</th>
      <td>78</td>
      <td>475</td>
      <td>931.0</td>
      <td>0.510204</td>
      <td>Brewmaster</td>
      <td>156</td>
      <td>0.167562</td>
    </tr>
    <tr>
      <th>67</th>
      <td>74</td>
      <td>6025</td>
      <td>11676.0</td>
      <td>0.516016</td>
      <td>Invoker</td>
      <td>1942</td>
      <td>0.166324</td>
    </tr>
    <tr>
      <th>108</th>
      <td>59</td>
      <td>1877</td>
      <td>3782.0</td>
      <td>0.496298</td>
      <td>Huskar</td>
      <td>626</td>
      <td>0.165521</td>
    </tr>
    <tr>
      <th>23</th>
      <td>19</td>
      <td>2783</td>
      <td>5305.0</td>
      <td>0.524599</td>
      <td>Tiny</td>
      <td>874</td>
      <td>0.164750</td>
    </tr>
    <tr>
      <th>13</th>
      <td>18</td>
      <td>1818</td>
      <td>3450.0</td>
      <td>0.526957</td>
      <td>Sven</td>
      <td>567</td>
      <td>0.164348</td>
    </tr>
    <tr>
      <th>28</th>
      <td>25</td>
      <td>4320</td>
      <td>8255.0</td>
      <td>0.523319</td>
      <td>Lina</td>
      <td>1354</td>
      <td>0.164022</td>
    </tr>
    <tr>
      <th>30</th>
      <td>76</td>
      <td>818</td>
      <td>1564.0</td>
      <td>0.523018</td>
      <td>Outworld Devourer</td>
      <td>256</td>
      <td>0.163683</td>
    </tr>
    <tr>
      <th>62</th>
      <td>3</td>
      <td>1319</td>
      <td>2553.0</td>
      <td>0.516647</td>
      <td>Bane</td>
      <td>415</td>
      <td>0.162554</td>
    </tr>
    <tr>
      <th>14</th>
      <td>46</td>
      <td>3183</td>
      <td>6042.0</td>
      <td>0.526812</td>
      <td>Templar Assassin</td>
      <td>979</td>
      <td>0.162032</td>
    </tr>
    <tr>
      <th>77</th>
      <td>110</td>
      <td>1557</td>
      <td>3029.0</td>
      <td>0.514031</td>
      <td>Phoenix</td>
      <td>486</td>
      <td>0.160449</td>
    </tr>
    <tr>
      <th>16</th>
      <td>32</td>
      <td>2179</td>
      <td>4140.0</td>
      <td>0.526329</td>
      <td>Riki</td>
      <td>662</td>
      <td>0.159903</td>
    </tr>
    <tr>
      <th>43</th>
      <td>2</td>
      <td>2395</td>
      <td>4601.0</td>
      <td>0.520539</td>
      <td>Axe</td>
      <td>734</td>
      <td>0.159531</td>
    </tr>
    <tr>
      <th>35</th>
      <td>111</td>
      <td>527</td>
      <td>1009.0</td>
      <td>0.522299</td>
      <td>Oracle</td>
      <td>160</td>
      <td>0.158573</td>
    </tr>
    <tr>
      <th>44</th>
      <td>21</td>
      <td>10867</td>
      <td>20881.0</td>
      <td>0.520425</td>
      <td>Windranger</td>
      <td>3294</td>
      <td>0.157751</td>
    </tr>
    <tr>
      <th>22</th>
      <td>39</td>
      <td>5556</td>
      <td>10590.0</td>
      <td>0.524646</td>
      <td>Queen of Pain</td>
      <td>1668</td>
      <td>0.157507</td>
    </tr>
    <tr>
      <th>3</th>
      <td>15</td>
      <td>1072</td>
      <td>2017.0</td>
      <td>0.531482</td>
      <td>Razor</td>
      <td>316</td>
      <td>0.156668</td>
    </tr>
    <tr>
      <th>45</th>
      <td>41</td>
      <td>1661</td>
      <td>3193.0</td>
      <td>0.520200</td>
      <td>Faceless Void</td>
      <td>498</td>
      <td>0.155966</td>
    </tr>
    <tr>
      <th>51</th>
      <td>86</td>
      <td>4246</td>
      <td>8183.0</td>
      <td>0.518881</td>
      <td>Rubick</td>
      <td>1271</td>
      <td>0.155322</td>
    </tr>
    <tr>
      <th>15</th>
      <td>91</td>
      <td>892</td>
      <td>1694.0</td>
      <td>0.526564</td>
      <td>Io</td>
      <td>263</td>
      <td>0.155254</td>
    </tr>
    <tr>
      <th>7</th>
      <td>37</td>
      <td>988</td>
      <td>1871.0</td>
      <td>0.528060</td>
      <td>Warlock</td>
      <td>290</td>
      <td>0.154997</td>
    </tr>
    <tr>
      <th>64</th>
      <td>11</td>
      <td>8784</td>
      <td>17007.0</td>
      <td>0.516493</td>
      <td>Shadow Fiend</td>
      <td>2634</td>
      <td>0.154877</td>
    </tr>
    <tr>
      <th>102</th>
      <td>79</td>
      <td>701</td>
      <td>1395.0</td>
      <td>0.502509</td>
      <td>Shadow Demon</td>
      <td>215</td>
      <td>0.154122</td>
    </tr>
    <tr>
      <th>91</th>
      <td>33</td>
      <td>1314</td>
      <td>2579.0</td>
      <td>0.509500</td>
      <td>Enigma</td>
      <td>396</td>
      <td>0.153548</td>
    </tr>
    <tr>
      <th>96</th>
      <td>53</td>
      <td>1698</td>
      <td>3344.0</td>
      <td>0.507775</td>
      <td>Nature's Prophet</td>
      <td>512</td>
      <td>0.153110</td>
    </tr>
    <tr>
      <th>84</th>
      <td>71</td>
      <td>3746</td>
      <td>7311.0</td>
      <td>0.512379</td>
      <td>Spirit Breaker</td>
      <td>1113</td>
      <td>0.152236</td>
    </tr>
    <tr>
      <th>56</th>
      <td>109</td>
      <td>804</td>
      <td>1551.0</td>
      <td>0.518375</td>
      <td>Terrorblade</td>
      <td>236</td>
      <td>0.152160</td>
    </tr>
    <tr>
      <th>69</th>
      <td>28</td>
      <td>5764</td>
      <td>11181.0</td>
      <td>0.515517</td>
      <td>Slardar</td>
      <td>1691</td>
      <td>0.151239</td>
    </tr>
    <tr>
      <th>53</th>
      <td>69</td>
      <td>4117</td>
      <td>7938.0</td>
      <td>0.518644</td>
      <td>Doom</td>
      <td>1194</td>
      <td>0.150416</td>
    </tr>
    <tr>
      <th>48</th>
      <td>96</td>
      <td>965</td>
      <td>1857.0</td>
      <td>0.519655</td>
      <td>Centaur Warrunner</td>
      <td>278</td>
      <td>0.149704</td>
    </tr>
    <tr>
      <th>21</th>
      <td>85</td>
      <td>3123</td>
      <td>5951.0</td>
      <td>0.524786</td>
      <td>Undying</td>
      <td>889</td>
      <td>0.149387</td>
    </tr>
    <tr>
      <th>12</th>
      <td>56</td>
      <td>1307</td>
      <td>2479.0</td>
      <td>0.527229</td>
      <td>Clinkz</td>
      <td>370</td>
      <td>0.149254</td>
    </tr>
    <tr>
      <th>5</th>
      <td>104</td>
      <td>4778</td>
      <td>9025.0</td>
      <td>0.529418</td>
      <td>Legion Commander</td>
      <td>1343</td>
      <td>0.148809</td>
    </tr>
    <tr>
      <th>90</th>
      <td>52</td>
      <td>625</td>
      <td>1226.0</td>
      <td>0.509788</td>
      <td>Leshrac</td>
      <td>182</td>
      <td>0.148450</td>
    </tr>
    <tr>
      <th>72</th>
      <td>13</td>
      <td>907</td>
      <td>1761.0</td>
      <td>0.515048</td>
      <td>Puck</td>
      <td>261</td>
      <td>0.148211</td>
    </tr>
    <tr>
      <th>85</th>
      <td>101</td>
      <td>1524</td>
      <td>2976.0</td>
      <td>0.512097</td>
      <td>Skywrath Mage</td>
      <td>441</td>
      <td>0.148185</td>
    </tr>
    <tr>
      <th>6</th>
      <td>63</td>
      <td>1356</td>
      <td>2566.0</td>
      <td>0.528449</td>
      <td>Weaver</td>
      <td>380</td>
      <td>0.148090</td>
    </tr>
    <tr>
      <th>63</th>
      <td>16</td>
      <td>1627</td>
      <td>3150.0</td>
      <td>0.516508</td>
      <td>Sand King</td>
      <td>464</td>
      <td>0.147302</td>
    </tr>
    <tr>
      <th>24</th>
      <td>100</td>
      <td>5406</td>
      <td>10306.0</td>
      <td>0.524549</td>
      <td>Tusk</td>
      <td>1518</td>
      <td>0.147293</td>
    </tr>
    <tr>
      <th>104</th>
      <td>77</td>
      <td>491</td>
      <td>985.0</td>
      <td>0.498477</td>
      <td>Lycan</td>
      <td>145</td>
      <td>0.147208</td>
    </tr>
    <tr>
      <th>94</th>
      <td>34</td>
      <td>1329</td>
      <td>2610.0</td>
      <td>0.509195</td>
      <td>Tinker</td>
      <td>384</td>
      <td>0.147126</td>
    </tr>
    <tr>
      <th>39</th>
      <td>1</td>
      <td>4899</td>
      <td>9396.0</td>
      <td>0.521392</td>
      <td>Anti-Mage</td>
      <td>1378</td>
      <td>0.146658</td>
    </tr>
    <tr>
      <th>76</th>
      <td>51</td>
      <td>2213</td>
      <td>4301.0</td>
      <td>0.514532</td>
      <td>Clockwerk</td>
      <td>630</td>
      <td>0.146478</td>
    </tr>
    <tr>
      <th>49</th>
      <td>9</td>
      <td>3746</td>
      <td>7210.0</td>
      <td>0.519556</td>
      <td>Mirana</td>
      <td>1055</td>
      <td>0.146325</td>
    </tr>
    <tr>
      <th>107</th>
      <td>103</td>
      <td>416</td>
      <td>838.0</td>
      <td>0.496420</td>
      <td>Elder Titan</td>
      <td>122</td>
      <td>0.145585</td>
    </tr>
    <tr>
      <th>41</th>
      <td>88</td>
      <td>1408</td>
      <td>2701.0</td>
      <td>0.521288</td>
      <td>Nyx Assassin</td>
      <td>393</td>
      <td>0.145502</td>
    </tr>
    <tr>
      <th>78</th>
      <td>17</td>
      <td>1237</td>
      <td>2407.0</td>
      <td>0.513918</td>
      <td>Storm Spirit</td>
      <td>349</td>
      <td>0.144994</td>
    </tr>
    <tr>
      <th>9</th>
      <td>67</td>
      <td>3513</td>
      <td>6660.0</td>
      <td>0.527477</td>
      <td>Spectre</td>
      <td>956</td>
      <td>0.143544</td>
    </tr>
    <tr>
      <th>73</th>
      <td>99</td>
      <td>2145</td>
      <td>4167.0</td>
      <td>0.514759</td>
      <td>Bristleback</td>
      <td>598</td>
      <td>0.143509</td>
    </tr>
    <tr>
      <th>36</th>
      <td>60</td>
      <td>1578</td>
      <td>3023.0</td>
      <td>0.521998</td>
      <td>Night Stalker</td>
      <td>433</td>
      <td>0.143235</td>
    </tr>
    <tr>
      <th>99</th>
      <td>45</td>
      <td>769</td>
      <td>1522.0</td>
      <td>0.505256</td>
      <td>Pugna</td>
      <td>218</td>
      <td>0.143233</td>
    </tr>
    <tr>
      <th>81</th>
      <td>93</td>
      <td>4322</td>
      <td>8426.0</td>
      <td>0.512936</td>
      <td>Slark</td>
      <td>1206</td>
      <td>0.143128</td>
    </tr>
    <tr>
      <th>31</th>
      <td>97</td>
      <td>1794</td>
      <td>3431.0</td>
      <td>0.522880</td>
      <td>Magnus</td>
      <td>489</td>
      <td>0.142524</td>
    </tr>
    <tr>
      <th>68</th>
      <td>35</td>
      <td>1964</td>
      <td>3809.0</td>
      <td>0.515621</td>
      <td>Sniper</td>
      <td>538</td>
      <td>0.141244</td>
    </tr>
    <tr>
      <th>42</th>
      <td>43</td>
      <td>897</td>
      <td>1721.0</td>
      <td>0.521209</td>
      <td>Death Prophet</td>
      <td>243</td>
      <td>0.141197</td>
    </tr>
    <tr>
      <th>27</th>
      <td>62</td>
      <td>3557</td>
      <td>6793.0</td>
      <td>0.523627</td>
      <td>Bounty Hunter</td>
      <td>954</td>
      <td>0.140439</td>
    </tr>
    <tr>
      <th>55</th>
      <td>44</td>
      <td>3775</td>
      <td>7280.0</td>
      <td>0.518544</td>
      <td>Phantom Assassin</td>
      <td>1022</td>
      <td>0.140385</td>
    </tr>
    <tr>
      <th>95</th>
      <td>8</td>
      <td>5279</td>
      <td>10394.0</td>
      <td>0.507889</td>
      <td>Juggernaut</td>
      <td>1451</td>
      <td>0.139600</td>
    </tr>
    <tr>
      <th>20</th>
      <td>36</td>
      <td>3137</td>
      <td>5969.0</td>
      <td>0.525549</td>
      <td>Necrophos</td>
      <td>829</td>
      <td>0.138884</td>
    </tr>
    <tr>
      <th>98</th>
      <td>6</td>
      <td>1322</td>
      <td>2608.0</td>
      <td>0.506902</td>
      <td>Drow Ranger</td>
      <td>362</td>
      <td>0.138804</td>
    </tr>
    <tr>
      <th>33</th>
      <td>68</td>
      <td>3529</td>
      <td>6753.0</td>
      <td>0.522583</td>
      <td>Ancient Apparition</td>
      <td>931</td>
      <td>0.137865</td>
    </tr>
    <tr>
      <th>0</th>
      <td>48</td>
      <td>1270</td>
      <td>2378.0</td>
      <td>0.534062</td>
      <td>Luna</td>
      <td>327</td>
      <td>0.137511</td>
    </tr>
    <tr>
      <th>75</th>
      <td>66</td>
      <td>298</td>
      <td>579.0</td>
      <td>0.514680</td>
      <td>Chen</td>
      <td>79</td>
      <td>0.136442</td>
    </tr>
    <tr>
      <th>58</th>
      <td>73</td>
      <td>5087</td>
      <td>9823.0</td>
      <td>0.517866</td>
      <td>Alchemist</td>
      <td>1335</td>
      <td>0.135906</td>
    </tr>
    <tr>
      <th>82</th>
      <td>31</td>
      <td>2403</td>
      <td>4687.0</td>
      <td>0.512695</td>
      <td>Lich</td>
      <td>636</td>
      <td>0.135694</td>
    </tr>
    <tr>
      <th>60</th>
      <td>57</td>
      <td>2670</td>
      <td>5161.0</td>
      <td>0.517342</td>
      <td>Omniknight</td>
      <td>695</td>
      <td>0.134664</td>
    </tr>
    <tr>
      <th>34</th>
      <td>92</td>
      <td>521</td>
      <td>997.0</td>
      <td>0.522568</td>
      <td>Visage</td>
      <td>134</td>
      <td>0.134403</td>
    </tr>
    <tr>
      <th>25</th>
      <td>26</td>
      <td>3870</td>
      <td>7382.0</td>
      <td>0.524248</td>
      <td>Lion</td>
      <td>989</td>
      <td>0.133975</td>
    </tr>
    <tr>
      <th>52</th>
      <td>7</td>
      <td>5873</td>
      <td>11323.0</td>
      <td>0.518679</td>
      <td>Earthshaker</td>
      <td>1515</td>
      <td>0.133798</td>
    </tr>
    <tr>
      <th>32</th>
      <td>81</td>
      <td>1225</td>
      <td>2344.0</td>
      <td>0.522611</td>
      <td>Chaos Knight</td>
      <td>312</td>
      <td>0.133106</td>
    </tr>
    <tr>
      <th>4</th>
      <td>23</td>
      <td>1348</td>
      <td>2543.0</td>
      <td>0.530083</td>
      <td>Kunkka</td>
      <td>335</td>
      <td>0.131734</td>
    </tr>
    <tr>
      <th>97</th>
      <td>29</td>
      <td>1217</td>
      <td>2400.0</td>
      <td>0.507083</td>
      <td>Tidehunter</td>
      <td>315</td>
      <td>0.131250</td>
    </tr>
    <tr>
      <th>57</th>
      <td>94</td>
      <td>1223</td>
      <td>2360.0</td>
      <td>0.518220</td>
      <td>Medusa</td>
      <td>307</td>
      <td>0.130085</td>
    </tr>
    <tr>
      <th>18</th>
      <td>54</td>
      <td>1359</td>
      <td>2585.0</td>
      <td>0.525725</td>
      <td>Lifestealer</td>
      <td>336</td>
      <td>0.129981</td>
    </tr>
    <tr>
      <th>71</th>
      <td>4</td>
      <td>1523</td>
      <td>2956.0</td>
      <td>0.515223</td>
      <td>Bloodseeker</td>
      <td>384</td>
      <td>0.129905</td>
    </tr>
    <tr>
      <th>29</th>
      <td>5</td>
      <td>4104</td>
      <td>7846.0</td>
      <td>0.523069</td>
      <td>Crystal Maiden</td>
      <td>1013</td>
      <td>0.129110</td>
    </tr>
    <tr>
      <th>47</th>
      <td>30</td>
      <td>3807</td>
      <td>7321.0</td>
      <td>0.520011</td>
      <td>Witch Doctor</td>
      <td>942</td>
      <td>0.128671</td>
    </tr>
    <tr>
      <th>40</th>
      <td>49</td>
      <td>1001</td>
      <td>1920.0</td>
      <td>0.521354</td>
      <td>Dragon Knight</td>
      <td>247</td>
      <td>0.128646</td>
    </tr>
    <tr>
      <th>2</th>
      <td>27</td>
      <td>1909</td>
      <td>3589.0</td>
      <td>0.531903</td>
      <td>Shadow Shaman</td>
      <td>461</td>
      <td>0.128448</td>
    </tr>
    <tr>
      <th>106</th>
      <td>90</td>
      <td>1077</td>
      <td>2167.0</td>
      <td>0.497000</td>
      <td>Keeper of the Light</td>
      <td>276</td>
      <td>0.127365</td>
    </tr>
    <tr>
      <th>10</th>
      <td>84</td>
      <td>2296</td>
      <td>4353.0</td>
      <td>0.527452</td>
      <td>Ogre Magi</td>
      <td>554</td>
      <td>0.127269</td>
    </tr>
    <tr>
      <th>80</th>
      <td>83</td>
      <td>894</td>
      <td>1742.0</td>
      <td>0.513203</td>
      <td>Treant Protector</td>
      <td>220</td>
      <td>0.126292</td>
    </tr>
    <tr>
      <th>17</th>
      <td>75</td>
      <td>3801</td>
      <td>7224.0</td>
      <td>0.526163</td>
      <td>Silencer</td>
      <td>908</td>
      <td>0.125692</td>
    </tr>
    <tr>
      <th>61</th>
      <td>112</td>
      <td>3981</td>
      <td>7697.0</td>
      <td>0.517214</td>
      <td>Winter Wyvern</td>
      <td>963</td>
      <td>0.125114</td>
    </tr>
    <tr>
      <th>65</th>
      <td>47</td>
      <td>1905</td>
      <td>3690.0</td>
      <td>0.516260</td>
      <td>Viper</td>
      <td>460</td>
      <td>0.124661</td>
    </tr>
    <tr>
      <th>1</th>
      <td>102</td>
      <td>1764</td>
      <td>3310.0</td>
      <td>0.532931</td>
      <td>Abaddon</td>
      <td>412</td>
      <td>0.124471</td>
    </tr>
    <tr>
      <th>109</th>
      <td>80</td>
      <td>465</td>
      <td>967.0</td>
      <td>0.480869</td>
      <td>Lone Druid</td>
      <td>120</td>
      <td>0.124095</td>
    </tr>
    <tr>
      <th>46</th>
      <td>64</td>
      <td>1429</td>
      <td>2748.0</td>
      <td>0.520015</td>
      <td>Jakiro</td>
      <td>339</td>
      <td>0.123362</td>
    </tr>
    <tr>
      <th>88</th>
      <td>72</td>
      <td>3499</td>
      <td>6856.0</td>
      <td>0.510356</td>
      <td>Gyrocopter</td>
      <td>845</td>
      <td>0.123250</td>
    </tr>
    <tr>
      <th>92</th>
      <td>55</td>
      <td>2149</td>
      <td>4219.0</td>
      <td>0.509362</td>
      <td>Dark Seer</td>
      <td>518</td>
      <td>0.122778</td>
    </tr>
    <tr>
      <th>70</th>
      <td>40</td>
      <td>1554</td>
      <td>3015.0</td>
      <td>0.515423</td>
      <td>Venomancer</td>
      <td>370</td>
      <td>0.122720</td>
    </tr>
    <tr>
      <th>66</th>
      <td>50</td>
      <td>4338</td>
      <td>8403.0</td>
      <td>0.516244</td>
      <td>Dazzle</td>
      <td>1030</td>
      <td>0.122575</td>
    </tr>
    <tr>
      <th>59</th>
      <td>42</td>
      <td>4036</td>
      <td>7794.0</td>
      <td>0.517834</td>
      <td>Wraith King</td>
      <td>952</td>
      <td>0.122145</td>
    </tr>
    <tr>
      <th>74</th>
      <td>87</td>
      <td>2445</td>
      <td>4750.0</td>
      <td>0.514737</td>
      <td>Disruptor</td>
      <td>540</td>
      <td>0.113684</td>
    </tr>
    <tr>
      <th>103</th>
      <td>38</td>
      <td>630</td>
      <td>1261.0</td>
      <td>0.499603</td>
      <td>Beastmaster</td>
      <td>141</td>
      <td>0.111816</td>
    </tr>
    <tr>
      <th>26</th>
      <td>12</td>
      <td>1912</td>
      <td>3650.0</td>
      <td>0.523836</td>
      <td>Phantom Lancer</td>
      <td>397</td>
      <td>0.108767</td>
    </tr>
    <tr>
      <th>79</th>
      <td>58</td>
      <td>522</td>
      <td>1016.0</td>
      <td>0.513780</td>
      <td>Enchantress</td>
      <td>105</td>
      <td>0.103346</td>
    </tr>
    <tr>
      <th>86</th>
      <td>20</td>
      <td>2146</td>
      <td>4194.0</td>
      <td>0.511683</td>
      <td>Vengeful Spirit</td>
      <td>407</td>
      <td>0.097043</td>
    </tr>
  </tbody>
</table>
</div>
</div>

</div>

</div>

</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell   ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[9]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="c1"># Masking code from https://stackoverflow.com/questions/25516325/non-overlapping-scatter-plot-labels-using-matplotlib</span>
<span class="n">random</span><span class="o">.</span><span class="n">seed</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
<span class="n">np</span><span class="o">.</span><span class="n">random</span><span class="o">.</span><span class="n">seed</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
<span class="n">n</span> <span class="o">=</span> <span class="mi">110</span>


<span class="n">labels</span> <span class="o">=</span> <span class="n">swears_per_hero</span><span class="p">[</span><span class="s1">&#39;localized_name&#39;</span><span class="p">]</span>
<span class="n">x</span> <span class="o">=</span> <span class="n">swears_per_hero</span><span class="p">[</span><span class="s1">&#39;hero_winrate&#39;</span><span class="p">]</span>
<span class="n">y</span> <span class="o">=</span> <span class="n">swears_per_hero</span><span class="p">[</span><span class="s1">&#39;profanities_per_play&#39;</span><span class="p">]</span>


<span class="n">fig</span><span class="p">,</span> <span class="n">ax</span> <span class="o">=</span> <span class="n">pl</span><span class="o">.</span><span class="n">subplots</span><span class="p">(</span><span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">20</span><span class="p">,</span> <span class="mi">10</span><span class="p">));</span>

<span class="n">linear_regressor</span> <span class="o">=</span> <span class="n">LinearRegression</span><span class="p">()</span>
<span class="n">linear_regressor</span><span class="o">.</span><span class="n">fit</span><span class="p">(</span><span class="n">x</span><span class="o">.</span><span class="n">values</span><span class="o">.</span><span class="n">reshape</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span><span class="mi">1</span><span class="p">),</span> <span class="n">y</span><span class="p">)</span>
<span class="n">x_pred</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">linspace</span><span class="p">(</span><span class="n">x</span><span class="o">.</span><span class="n">min</span><span class="p">(),</span> <span class="n">x</span><span class="o">.</span><span class="n">max</span><span class="p">(),</span> <span class="n">num</span><span class="o">=</span><span class="mi">200</span><span class="p">)</span><span class="o">.</span><span class="n">reshape</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
<span class="n">y_pred</span> <span class="o">=</span> <span class="n">linear_regressor</span><span class="o">.</span><span class="n">predict</span><span class="p">(</span><span class="n">x_pred</span><span class="p">)</span>  
<span class="n">ax</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">x_pred</span><span class="p">,</span> <span class="n">y_pred</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="s2">&quot;#FFCCCC&quot;</span><span class="p">,</span> <span class="n">lw</span><span class="o">=</span><span class="mi">4</span><span class="p">)</span>

<span class="n">ax</span><span class="o">.</span><span class="n">scatter</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s1">&#39;Winrate&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylabel</span><span class="p">(</span><span class="s1">&#39;Profanity rate&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">title</span><span class="p">(</span><span class="s2">&quot;Winrate vs Profanities&quot;</span><span class="p">)</span>



<span class="n">ann</span> <span class="o">=</span> <span class="p">[]</span>
<span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="n">n</span><span class="p">):</span>
    <span class="n">ann</span><span class="o">.</span><span class="n">append</span><span class="p">(</span><span class="n">ax</span><span class="o">.</span><span class="n">annotate</span><span class="p">(</span><span class="n">labels</span><span class="p">[</span><span class="n">i</span><span class="p">],</span> <span class="n">xy</span> <span class="o">=</span> <span class="p">(</span><span class="n">x</span><span class="p">[</span><span class="n">i</span><span class="p">]</span> <span class="o">+</span> <span class="mf">.0002</span><span class="p">,</span> <span class="n">y</span><span class="p">[</span><span class="n">i</span><span class="p">]</span><span class="o">-</span><span class="mf">.0005</span><span class="p">)))</span>

<span class="n">mask</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="n">fig</span><span class="o">.</span><span class="n">canvas</span><span class="o">.</span><span class="n">get_width_height</span><span class="p">(),</span> <span class="nb">bool</span><span class="p">)</span>

<span class="n">fig</span><span class="o">.</span><span class="n">canvas</span><span class="o">.</span><span class="n">draw</span><span class="p">()</span>

<span class="k">for</span> <span class="n">a</span> <span class="ow">in</span> <span class="n">ann</span><span class="p">:</span>
    <span class="n">bbox</span> <span class="o">=</span> <span class="n">a</span><span class="o">.</span><span class="n">get_window_extent</span><span class="p">()</span>
    <span class="n">x0</span> <span class="o">=</span> <span class="nb">int</span><span class="p">(</span><span class="n">bbox</span><span class="o">.</span><span class="n">x0</span><span class="p">)</span>
    <span class="n">x1</span> <span class="o">=</span> <span class="nb">int</span><span class="p">(</span><span class="n">math</span><span class="o">.</span><span class="n">ceil</span><span class="p">(</span><span class="n">bbox</span><span class="o">.</span><span class="n">x1</span><span class="p">))</span>
    <span class="n">y0</span> <span class="o">=</span> <span class="nb">int</span><span class="p">(</span><span class="n">bbox</span><span class="o">.</span><span class="n">y0</span><span class="p">)</span>
    <span class="n">y1</span> <span class="o">=</span> <span class="nb">int</span><span class="p">(</span><span class="n">math</span><span class="o">.</span><span class="n">ceil</span><span class="p">(</span><span class="n">bbox</span><span class="o">.</span><span class="n">y1</span><span class="p">))</span>

    <span class="n">s</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">s_</span><span class="p">[</span><span class="n">x0</span><span class="p">:</span><span class="n">x1</span><span class="o">+</span><span class="mi">1</span><span class="p">,</span> <span class="n">y0</span><span class="p">:</span><span class="n">y1</span><span class="o">+</span><span class="mi">1</span><span class="p">]</span>
    <span class="k">if</span> <span class="n">np</span><span class="o">.</span><span class="n">any</span><span class="p">(</span><span class="n">mask</span><span class="p">[</span><span class="n">s</span><span class="p">]):</span>
        <span class="n">a</span><span class="o">.</span><span class="n">set_visible</span><span class="p">(</span><span class="kc">False</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">mask</span><span class="p">[</span><span class="n">s</span><span class="p">]</span> <span class="o">=</span> <span class="kc">True</span>
</pre></div>

     </div>
</div>
</div>
</div>

<div class="jp-Cell-outputWrapper">
<div class="jp-Collapser jp-OutputCollapser jp-Cell-outputCollapser">
</div>


<div class="jp-OutputArea jp-Cell-outputArea">

<div class="jp-OutputArea-child">

    
    <div class="jp-OutputPrompt jp-OutputArea-prompt"></div>




<div class="jp-RenderedImage jp-OutputArea-output ">
<img src="data:image/png;base64,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nUne9++IbM8qh/PjSV/i9kPGPa8lfZYUQIOM4wTSZ6q9Y1lW2FWMRURERIqZUX5EERERERERERFxlGYqiYiIiIiIiIiIwxRUEhERERERERERhymoJCIiIiIiIiIiDlNQSUREREREREREHFa6uAdQmFxdXS13d/fiHoaIiIiIiIiIyA0jKirqpGVZNS8tv6GCSu7u7kRGRhb3MEREREREREREbhjGmMM5lWv5m4iIiIiIiIiIOExBJRERERERERERcZiCSiIiIiIiIiIi4jAFlURERERERERExGEKKomIiIiIiIiIiMMUVBIREREREREREYcpqCQiIiIiIiIiIg5TUElERERERERERBymoJKIiIiIiIiIiDhMQSUREREREREREXGYgkoiIiIiIiIiIuIwBZVERERERERERMRhCiqJiIiIiIiIiIjDFFQSERERERERERGHKagkIiIiIiIiIiIOU1BJREREREREREQcpqCSiIiIiIiIiIg4TEElERERERERERFxmIJKIiJSYhljGDRokP04JSWFmjVr0rVr12IclYiIiIiIgIJKIiJSglWsWJGYmBgSExMB+Pbbb3FzcyvmUYmIiIiICCioJCIiJcCy7XEETVuHx4RVBE1bx7LtcfZzd999N6tWrQJg/vz59O/f337ur7/+4qGHHqJZs2b4+vqyfPlyAFJTUxk3bhzNmjXD29ubd999F4CwsDDatGlDz549ady4MSNGjCAtLc3et5eXF56enowfP/5a3bqIiIiIyHVLQSURESlWy7bHMXHJLuLiE7GAuPhEJi7ZZQ8s9evXjwULFpCUlMTOnTtp0aKFve1LL71E+/bt2bp1K+vXr2fcuHH89ddffPDBB1SpUoWtW7eydetWZs+ezS+//ALAli1bePXVV9m1axeHDh1iyZIlHD16lPHjx7Nu3Tqio6PZunUry5YtK4anISIiIiJy/Shd3AMQEZGb2/TV+0hMTs1WlpicyvTV+wDw9vYmNjaW+fPnc88992Srt2bNGr788ktmzJgBQFJSEr/++itr1qxh586dLF68GIAzZ85w4MABypYtS/Pmzalfvz4A/fv3Z9OmTZQpU4bg4GBq1qwJwMCBA9mwYQM9evQoylsXEREREbmuKagkIiLF6mh8Yr7l3bp145lnniEsLIxTp07Zyy3L4osvvqBhw4bZ2lqWxZtvvklISEi28rCwMIwx2cqMMViWdbW3ISIiIiJy09HyNxERKVZ1qjrnW/7QQw8xadIkvLy8stUJCQnhzTfftAeFtm/fbi//3//+R3JyMgD79+/nr7/+AtKXv/3yyy+kpaWxcOFCWrVqRYsWLfj+++85efIkqampzJ8/n7Zt2xb6vYqIiIiI3EgUVBIRkWI1LqQhzmWcspU5l3FiXMjfs4/q1q3LmDFjLmv7/PPPk5ycjLe3N56enjz//PMAPPLIIzRu3Bg/Pz88PT159NFHSUlJASAwMJAJEybg6emJh4cHPXv2pHbt2kydOpV27drh4+ODn58f3bt3L8K7FhERERG5/pkbacq/v7+/FRkZWdzDEBERBy3bHsf01fs4Gp9InarOjAtpSA9ft0K/TlhYGDNmzGDlypWF3reIiIiIyI3KGBNlWZb/peXKqSQiIsWuh69bkQSRRERERESk6CioJCIiN43g4GCCg4OLexgiIiIiIjcE5VQSERERERERERGHKagkIiIiIiIiIiIOU1BJREREREREREQcpqCSiIiIiIiIiIg4TEElERERERERERFxmIJKIiIiIiIiIiLiMAWVRERERERERETEYQoqiYiIiIiIiIiIwxRUEhERERERERERhymoJCIiIiIiIiIiDlNQSUREREREREREHKagkoiIiIiIiIiIOExBJRERERERERERcZiCSiIiIiIiIiIi4jAFlURERERERERExGEKKomIiIiIiIiIiMMUVBIREREREREREYcVaVDJGNPZGLPPGHPQGDMhh/MDjTE7M77CjTE+GeW3GWPWG2N+MsbsNsaMKcpxioiIiIiIiIiIY0oXVcfGGCfgbaAj8Buw1RjzpWVZe7JU+wVoa1nWn8aYu4H3gBZACvC0ZVnbjDGVgChjzLeXtBURERERERERkWJSlDOVmgMHLcv62bKsi8ACoHvWCpZlhVuW9WfG4Q9A3YzyY5Zlbct4fQ74CXArwrGKiIiIiIiIiIgDijKo5AYcyXL8G3kHhh4Gvr600BjjDvgCP+bUyBgz3BgTaYyJPHHixJWPVkRERERERERECqwog0omhzIrx4rGtCM9qDT+knIX4AvgScuyzubU1rKs9yzL8rcsy79mzZpXOWQRERERERERESmIIsupRPrMpNuyHNcFjl5ayRjjDbwP3G1Z1qks5WVIDyjNsyxrSRGOU0REREREREREHFSUM5W2Ag2MMR7GmLJAP+DLrBWMMbcDS4BBlmXtz1JugA+AnyzLeq0IxygiIiIiIiIiIlegyGYqWZaVYowZBawGnIAPLcvabYwZkXF+FjAJqAG8kx5HIsWyLH8gCBgE7DLGRGd0+axlWV8V1XhFRERERERERKTgjGXlmObouuTv729FRkYW9zBERERERERERG4YxpiojElA2RTl8jcREREREREREblBKagkIiIiIiIiIiIOU1BJREREREREREQcpqCSiIiIiIiIiIg4TEElERERERERERFxmIJKIiIiIiIiIiLiMAWVRERERERERETEYQoqiYiIiIiIiIiIwxRUEhERERERERERhymoJCIiIiIiIiIiDlNQSUREREREREREHKagkoiIiIiIiIiIOExBJRERERERERERcZiCSiIiIiIiIiIi4jAFlURERERERERExGEKKomIiIiIiIiIiMMUVBIREREREREREYcpqCQiIiIiIiIiIg5TUElERERERERERBymoJKIiIiIiIiIiDhMQSUREREREREREXGYgkoiIiIiIiIiIuIwBZVERERERERERMRhCiqJiIiIiIiIiIjDFFQSERERERERERGHKagkIiIiIiIiIiIOU1BJREREREREREQcpqCSiIiIiIiIiIg4TEElERERERERERFxmIJKIiIiIiIiIiLiMAWVRERERERERETEYQoqiYiIiIiIiIiIwxRUEhERERERERERhymoJCIiIiIiIiIiDlNQSUREREREREREHKagkoiIiIiIiIiIOExBJRERERERERERcZiCSiIiIiIiIiIi4jAFlURERERERERExGEKKomIiIiIiIiIiMMUVBIREREREREREYcpqCQiIiIiIiIiIg5TUElERERERERERBymoJKIiIiIiIiIiDhMQSUREREREREREXGYgkoiIiIiIiIiIuIwBZVERERERERERMRhCiqJiIiIiIiIiIjDFFQSERERERERERGHKagkIiIiIiIiIiIOU1BJREREREREREQcpqCSiIiIiIiIiIg4TEElERERERERERFxmIJKIiIiIiIiIiLiMAWVRERERERERETEYQoqiYiIiIiIiIiIwxRUEhERERERERERhymoJCIiIiIiIiIiDlNQSUREREREREREHKagkoiIiIiIiIiIOExBJRERERERERER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zZtTt25dAGw2G7GxsdmCSj/88AN79uwhKCgIgIsXLxIYGEjlypUpX748jzzyCF26dMk1N5OIyI1AQSUREREREbnuTF+9zx5QypSYnMr01fsuCypZlkWTJk2IiIjIVn727Nlc+y9Xrpz9tZOTEykpKZf12bFjR+bPn39Z2y1btvDdd9+xYMEC3nrrLdatW1fg+xIRuZ4op5KIiIiIiFx3jsYnFri8XLlynDhxwh5USk5OZvfu3VSuXJm6deuybNkyIH1G0/nz5wt0/YCAADZv3szBgwcBOH/+PPv37ychIYEzZ85wzz33MHPmTHvCcBGRG5GCSiIiIiIict2pU9W5wOWlSpVi8eLFjB8/Hh8fH2w2G+Hh4QB88sknvPHGG3h7e9OyZUt+//33Al2/Zs2azJkzh/79++Pt7U1AQAB79+7l3LlzdO3aFW9vb9q2bct///vfK79JEZESzmTuYnAj8Pf3tzJ3YhARERERkRvXpTmVAJzLODG1l1eOybpFROTKGWOiLMvyv7RcOZVEREREROS6kxk4KsjubyIiUjQUVBIRERERketSD183BZFERIqRciqJiIiIiIiIiIjDFFQSERERERERERGHKagkIiIiIiIiIiIOU1BJREREREREREQcpqCSiIiIiIiIiIg4TEElEREREcnGyckJm81m/5o2bZpD7ZctW8aePXvsx8HBwURGRubZJi0tjdGjR+Pp6YmXlxfNmjXjl19+ybPNpEmTWLt2bb7nZs6cyfnz5x26BxEREclf6eIegIiIiIiULM7OzkRHR19R25SUFJYtW0bXrl1p3LhxgdstXLiQo0ePsnPnTkqVKsVvv/1GxYoV82wzZcqUHMtTU1OznZs5cyYPPPAAFSpUKPB4REREJH+aqSQiIiJyE1q2PY6gaevwmLCKoGnrWLY9Lt82U6ZMoVmzZnh6ejJ8+HAsywLSZyI9++yztG3blpdffpkvv/yScePGYbPZOHToEACLFi2iefPm3HnnnWzcuPGyvo8dO0bt2rUpVSr919O6detSrVo1AFxcXHj66afx8/OjQ4cOnDhxAoChQ4eyePFiANzd3ZkyZQqtWrVi0aJF9nNvvPEGR48epV27drRr1+7qH5yIiIjYKagkIiIicpNZtj2OiUt2ERefiAXExScycckue2ApMTEx2/K3hQsXAjBq1Ci2bt1KTEwMiYmJrFy50t5nfHw833//Pc899xzdunVj+vTpREdHc8cddwDpM5i2bNnCzJkzmTx58mVjuv/++1mxYgU2m42nn36a7du328/99ddf+Pn5sW3bNtq2bZtje4Dy5cuzadMm+vXrZy8bPXo0derUYf369axfv/6qn52IiIj8TUElERERue7Fxsbi6emZrSw0NJQZM2YU04hKtumr95GYnJqtLDE5lemr9wF/L3/L/Orbty8A69evp0WLFnh5ebFu3Tp2795tb59ZJze9evUCoGnTpsTGxl52vm7duuzbt4+pU6dSqlQpOnTowHfffQdAqVKl7P0/8MADbNq0Kcdr5DcGERERKVzKqSQiIiI3jZSUFEqX1q8/R+MTHSoHSEpKYuTIkURGRnLbbbcRGhpKUlKS/Xx++Y/KlSsHpCcBT0lJybXO3Xffzd13380tt9zCsmXL6NChw2X1jDE5ts9vDCIiIlK4NFNJRERErgtXkgMIsuf7ef3111m0aBGenp74+PjQpk0bIH2mU+vWrfHz88PPz4/w8PCivJViV6eqs0PlgD2A5OrqSkJCgj2XUU4qVarEuXPnHBrTtm3bOHr0KJC+E9zOnTupV6+e/Tjzep999hmtWrVyqO8rGY+IiIjkTx/ViYiISImXmQMoc8lWZg4ggB6+bvm2z8z3A+Dl5cXq1atxc3MjPj4egFq1avHtt99Svnx5Dhw4QP/+/YmMjCyamykBxoU0zPY8AZzLODEupCHwd06lTJ07d2batGkMGzYMLy8v3N3dadasWa799+vXj2HDhvHGG2/kGXzK6o8//mDYsGFcuHABgObNmzNq1CggfQbS7t27adq0KVWqVLHneCqo4cOHc/fdd1O7dm3lVRIRESlEJnPXjhuBv7+/dSP/AigiInKzCpq2jrgclma5VXVm84T2HD58mC5duhATE2M/FxoaSqVKlVixYgWTJ0+mbdu2AIwYMYJDhw5x//3306tXL2rUqMGZM2cYNWoU0dHRODk5sX//fs6fP3/N7q84LNsex/TV+zgan0idqs6MC2lYoABdcXBxcSEhIaG4hyE3gVOnTtmXXP7+++84OTlRs2ZNALZs2ULZsmVzbTt06FC6du1K7969CQ4OZsaMGfj7+9vPL1++nI8++ohly5YBMHXqVD744AMOHjwIwIoVK5g9ezZffvllgcYaFhbGjBkzsiXMd5S7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DBcuXMDJyYnSpUsTERHBY489lutW9CJy4zPGRFmWddk6cM1UEhERERERuY5ci4TPv/76K/fffz9paWmULVuW2bNnF+n1ROT6pKCSiIiIiIiIZNOgQQO2b99e3MMQkRJOibpFRERERERERMRhCiqJiIiIiIiIiIjDFFQSERERERERERGHKagkIiIiIiIiIiIOU1BJREREREREREQcpqCSiIiIiIiIiIg4TEElERERERERERFxmIJKIiIiIiIiIiLiMAWVRERERERERETEYQoqiYiIiIiIiIiIw4o0qGSM6WyM2WeMOWiMmZDD+UbGmAhjzAVjzDOXnBtrjNltjIkxxsw3xpQvyrGKiIiIiIiIiEjBFVlQyRjjBLwN3A00BvobYxpfUu00MBqYcUlbt4xyf8uyPAEnoF9RjVVERERERERERBxTlDOVmgMHLcv62bKsi8ACoHvWCpZl/WFZ1lYgOYf2pQFnY0xpoAJwtAjHKiIiIiIiIiIiDijKoJIbcCTL8W8ZZfmyLCuO9NlLvwLHgDOWZa3Jqa4xZrgxJtIYE3nixImrHLKIiIiIiIiIiBREUQaVTA5lVoEaGlON9FlNHkAdoKIx5oGc6lqW9Z5lWf6WZfnXrFnzigcrIiIiIiIiIiIFV5RBpd+A27Ic16XgS9juAn6xLOuEZVnJwBKgZSGPT0RERERERERErlBRBpW2Ag2MMR7GmLKkJ9r+soBtfwUCjDEVjDEG6AD8VETjFBERERERERERB5Uuqo4ty0oxxowCVpO+e9uHlmXtNsaMyDg/yxhzKxAJVAbSjDFPAo0ty/rRGLMY2AakANuB94pqrCIiIiIiIiIi4hhjWQVKc3Rd8Pf3tyIjI4t7GCIiIiIiIiIiNwxjTJRlWf6Xlhfl8jcREREREREREblBKagkIiIiIiIiIiIOU1BJREREREREREQcpqCSiIiIiIiIiIg4TEElERERERERERFxmIJKIiIiIiIiIiLiMAWVRERERERERETEYQoqiYiIiIiIiIiIwxRUEhERERERERERhymoJCIiIiIiIiIiDlNQSUREREREREREHKagkoiIiIiIiIiIOExBJRERERERERERcZiCSiIiIiIiIiIi4jAFlURERERERERExGEKKomIiMh1ycnJCZvNho+PD35+foSHhxfJdYKDg4mMjLzqfqKjo/nqq6/sx6Gh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2rBldDMFlCRbJq7Zn6rAOkDctUQmrtkPgJ2dHQ0aNOCLL77I9j47duwIQN26dVMVWA8ODub9999n1apVqQJKb7/9NufPn2fGjBkYRvKyhYsXL8bb2xsvLy/27t1LVFTUrZ6i3AYFlURERERyScqvucfPx2Hyv19zswospRg3bhyNGjVi586dtGvXjj///BOA33//nUWLFrFlyxYiIiKwtbVl/vz5AFy+fBl/f3927dpFQEAAs2bNyvI4O3fuZMqUKURFRXH48GG2bNnC8OHDqVixIsHBwQQHB3P27FnGjx/PunXrCA8Px8fHh8mTJ1v7KF68OJs3b6Zbt245f6BERKTQOJFOLa4b221sbFi8eDE7duzg3XffBaBIkSIkJf1vVb/4+PhUfYsVKwaAra0tCQkJVnv16tW5dOkSBw4cSLW9r68vYWFhnDt3DoAjR44wadIk1q9fT2RkJG3atElzDLkzFFQSERERySVZ/ZqblY0bN/Lkk08C0KZNG+tX2vXr1xMWFoavry+enp6sX7+ew4cPA1C0aFEr0+nmX3wz4ufnR+XKlbGxscHT0zPdPtu2bSMqKoqGDRvi6enJ3LlzOXr0qHV/165ds3VOInnJMAyeeuop63ZCQgL333+/9ZrIrsDAQLSKtEj6MirmfmO7g4MDK1euZP78+XzxxRc88MADnD59mn/++YcrV66wcuXKbB2ratWqLFu2jF69erF3716rvXXr1owePZo2bdpw6dIlLl68SIkSJShdujSnTp1i9erVt3eScstUU0lEREQkl2T1a252pKT138g0TXr37s17772X5j47Ozurz42/+N74K7Fpmly9etXqk/IL8c19bj5mixYtWLBgQbrjLFGiRLbPSSSvlChRgj179hAXF4e9vT0///wzlSppSpdIbhrVqlaqmkqQfpH3++67j59++omAgACcnJx48803qVevHtWqVct2rSWAWrVqMX/+fDp37swPP/xgtXfu3JlLly7Rrl07fvzxR7y8vHBxcaF69eo0bNjw9k9UbomCSiIiIiK55HaX7A4ICGD+/Pm8/vrrrF69mn///ReA5s2b0759e1544QXKly/PuXPnuHTpElWrVs1wX87OzoSFhdGlSxe+++47rl27luXxS5YsyaVLl3BycsLf358hQ4Zw6NAh/vOf/xAbG8tff/1FzZo1s3UuIrllxc7j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W9KrV69U4zFNk1GjRuHq6oqbm5u1JHG7du24fPky9erVs9og+UPk9OnT+eijj/D09LRSnjdu3EiDBg2oXr16qqyliRMn4uvri7u7O2PGjMn24zhnzhyGDh0KJM/L9/f3x9fXlzfffDPVL4sxMTFp5sNPnTqVEydO0LRpU5o2bZrtY4qISFrZXYWuoB8jO3JjSmVUVBT/+c9/aN68OTVq1MiDURYeGX0m2rBhA8WLF6dv375A8hf6jz76iNmzZxMbG0tsbCxdunSx6vrUq1fPWoHM0dGRN998k3r16rF169YMP0PdrFu3blbgbNmyZVb2DiR/lmjevLlVd+i7776z7nv77bepXbs2LVq0oHv37tb0sdvN0r4VGQVZy147a2XvAERERFC1atVM93XhwgUqVUq+vm/MaspoKmS9evX45ZdfOHv2LImJiSxYsIAmTZpYfSZNmkRAQABeXl4EBwdTrFgxSpcunea4q1atsgJUBw8exNbWljJlymQ6zludEprSv0KFCtjY2GQ4/fN2KKgkIiK5KqtCo1euXKF9+/asWLGC2rVrA/D777+zaNEitmzZQkREBLa2tsyfPz/LZXxLlSrF9u3bGTp0KM8//zwAzz33HC+88AI7duxg6dKlPPPMM9b2YWFhfPfdd3zzzTepxrxs2TJreeN169YxatQoTp48yffff28tc9y1a1dre2dnZwYNGsQLL7xARESEldp88uRJNm/ezMqVKxk9ejSQHBQ7ePAg27dvJyIigrCwMDZu3Jjjx/W5557jueeeY8eOHWmmGqQ3H3748OFUrFiR4OBggoODc3w8ERH5nztRFP1uKrxep04dDh8+nGGWxr0is89Ee/fupW7duqm2L1WqFA899BCHDh3is88+o2zZskRGRvLGG2+kKpJ++fJlXF1d+e233yhXrly6n6HS07x5czZu3EhiYiILFy5M9dmmePHiLF++nPDwcIKDg3nxxRcxTZPQ0FCWLl3Kzp07WbZsmRXYyi8ZBV97epend+/e1KlTB3d3d6Kiohg7dmym+3rppZd45ZVXaNiwYapAS0ZTIStUqMB7771H06ZN8fDwwNvbm/bt2wPQuHFjjh07RkBAALa2tlSpUiXdIt0AX331FbVq1cLT05OnnnqK+fPnZ5ptdjtTQiF70z9vh6a/iYhIrsqs0GiQVyXs7Oxo0KABX3zxBR9/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iIuLo1OnTtb3559++onnn38eJyenVPWo/vnnH7p3786ZM2fw8/PjxtJEkydPtupnPvPMMzz//PNER0fz2GOP0ahRI3799VcqVarEd999h719/gZClalUgGT061BGIiIi8Pf3x93dnQ4dOvDvv/8CyfPeX375Zfz8/KhZs6a11HViYiKjRo3C19cXd3d365eizJQvX56ZM2cybdo0TNNkzpw5dO7cmccff5yWLVumSd8bOnQoc+bMscaRUuPqyy+/pGbNmjRp0oQtW7bcysMjIiIiInlk4pr9qb5cA8RdS2Timv1A6ilwCxcupHv37mn20aVLF2xsbKha/T8UKfMg1/75i/gj4Vzes4ETXw7jzDcj+eeffzh48CAAfn5+CihJvrmXshAlfVlN1V2xYgWtW7emZs2a3HfffYSHhwPJAfEPP/yQ3bt388cff7Bs2TIA3nnnHUJDQ4mMjOSXX34hMjKS+Ph4+vfvzw8//MCmTZv4+++/reOPGzeORo0asXPnTtq1a8eff/4JJC/O8uWXX/Lbb7+xbds2Zs2axc6dOwE4ePAgQ4YMYe/evZQpU4alS5fewUcsfQoqFRA5nXsO0KtXL95//30iIyNxc3NLlUmUkJDA9u3bmTJlitX+xRdfULp0aXbs2MGOHTuYNWsWR44cyXJs1atXJykpidOnTwOwdetW5s6dy4YNG7J1bidPnmTMmDFs2bKFn3/+Od0pcSIiInJ3O3XqFD169KB69erUrVuX+vXrs3z58vwelmXKlCnExsbm9zDyTVb1ZYKCgli/fj3h4eHExcWlu/pTSubSqFa1sDEMuJ7IdN+jA3l4wGfMX/ULR44csTKVSpQokQdnkj8yK1URHR2Nq6trmvbQ0FCGDx+el8OSLAR5VWLL6GYcmdCGLaObKaB0j8kqmL5gwQK6desGQLdu3ViwYAGQHBCvXr06tra2dO/enc2bNwOwePFivL298fLyYu/evURFRbFv3z6qVatGjRo1MAyDJ5980jrWxo0brdtt2rShbNmyAGzevJkOHTpQokQJHB0d6dixo5UoUq1aNTw9PQGoW7cu0dHRefPg5ICCSgVEVhf0zS5cuMD58+dp0qQJAL1792bjxo3W/R07dgRSX2hr165l3rx5eHp6Uq9evVS/FGXlxlS8Fi1acN9992X73H777TcCAwO5//77KVq0KF27ds12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lUREREREcsHENfutgFKKuGuJTFyznyCvSplOf+vYsSMAdevWZdmyZQCsW7eOhQsXWtuULVsWgKJFi9K2bVtr+59//hmAv/76i65du3Ly5EmuXr1KtWrVrL5t2rShWLFiFCtWjPLly1tZTDfz8/Oz+q1du5bIyEiWLFkCwIULFzh48CC+vr7069ePa9euERQUhKenJ9WrV+fw4cMMGzaMNm3a0LJlyxw9diIiUjhp+puIiIiISC44cT4uR+03KlasGJBczDshIQEA0zQxDCPNtnZ2dlb7jdsPGzaMoUOHsnv3bmbMmEF8fHya/d/c52YlSpSw/jZNk08++YSIiAgiIiI4cuQILVu2JCAggI0bN1KpUiWeeuop5s2bR9myZdm1axeBgYF8+umnPPPMM1mes4iIFH4KKomIiIiI5IKKZexz1J6Vli1bMm3aNOt2yvS3jFy4cIFKlSoBMHfu3Cz3X7JkSS5dupTh/a1ateLzzz/n2rVrABw4cIDLly9z9OhRypcvT//+/Xn66acJDw/n7NmzJCUl8cQTT/D2228THh6enVMUEZFCTkElEREREZFcMKpVLeztUq+YZm9ny6hWtQCsmkop/40ePTrT/b3++uv8+++/uLq64uHhQXBwcKbbjx07ls6dO9O4cWOcnJyyHG+5cuVo2LAhrq6ujBo1Ks39zzzzDHXq1MHb2xtXV1cGDhxIQkICISEheHp64uXlxdKlS3nuuec4fvw4gYGBeHp60qdPH957770sjy8iIoWfYZpmfo8h1/j4+JihoaH5PQwRERERuUet2HmciWv2c+J8HBXL2DOqVS2CvCrl97BERERui2EYYaZp+tzcrkLdIiIiIiK5JMirkoJIIiJyz9D0NxERERERERERyTEFlUREREREREREJMcUVBIRERERERERkRxTUElERERERERERHJMQSUREREREREREcmxPA0qGYbR2jCM/YZhHDIMY3Q699c2DGOrYRhXDMMYmZO+IiIiIiIiIiKSf/IsqGQYhi3wKfAYUAfobhhGnZs2OwcMBybdQl8REREREREREckneZmp5AccMk3zsGmaV4GFQPsbNzBN87RpmjuAazntKyIiIiIiIiIi+Scvg0qVgGM33P7reluu9jUMY4BhGKGGYYSeOXPmlgYqIiIiIiIiIiI5k5dBJSOdNjO3+5qmOdM0TR/TNH3u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
"
>
</div>

</div>

</div>

</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell   ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[11]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">fig</span><span class="p">,</span> <span class="n">ax</span> <span class="o">=</span> <span class="n">plt</span><span class="o">.</span><span class="n">subplots</span><span class="p">(</span><span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">12</span><span class="p">,</span> <span class="mi">6</span><span class="p">));</span>

<span class="n">ax</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="n">swears_per_hero</span><span class="p">[</span><span class="s1">&#39;profanities_per_play&#39;</span><span class="p">],</span> <span class="n">labels</span><span class="o">=</span><span class="p">[</span><span class="s1">&#39;&#39;</span><span class="p">]</span> <span class="p">,</span>
                <span class="n">notch</span> <span class="o">=</span><span class="s1">&#39;True&#39;</span><span class="p">,</span> <span class="n">vert</span> <span class="o">=</span> <span class="mi">0</span><span class="p">)</span>

<span class="n">plt</span><span class="o">.</span><span class="n">title</span><span class="p">(</span><span class="s2">&quot;Profanities of Heroes&quot;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s1">&#39;Profanity rate&#39;</span><span class="p">)</span>

<span class="n">ax</span>
<span class="c1">#both Batrider and Meepo seem to be significant outliers</span>
</pre></div>

     </div>
</div>
</div>
</div>

<div class="jp-Cell-outputWrapper">
<div class="jp-Collapser jp-OutputCollapser jp-Cell-outputCollapser">
</div>


<div class="jp-OutputArea jp-Cell-outputArea">

<div class="jp-OutputArea-child">

    
    <div class="jp-OutputPrompt jp-OutputArea-prompt">Out[11]:</div>




<div class="jp-RenderedText jp-OutputArea-output jp-OutputArea-executeResult" data-mime-type="text/plain">
<pre>&lt;AxesSubplot:title={&#39;center&#39;:&#39;Profanities of Heroes&#39;}, xlabel=&#39;Profanity rate&#39;&gt;</pre>
</div>

</div>

<div class="jp-OutputArea-child">

    
    <div class="jp-OutputPrompt jp-OutputArea-prompt"></div>




<div class="jp-RenderedImage jp-OutputArea-output ">
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAq8AAAGDCAYAAAAF5/lNAAAAOXRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjUuMSwgaHR0cHM6Ly9tYXRwbG90bGliLm9yZy/YYfK9AAAACXBIWXMAAAsTAAALEwEAmpwYAAAcQklEQVR4nO3de7ScdX3v8c83IQEFERWxIsWoLQpySNBwUfASRRRRcXXhsWBjxIi19UhFPWpNcXGqWFc91kuXVuSi9ZZSEIEDVq2aAwblGggqWrxQbyBeg+ABDOF3/piJ3WICe2cne/Lb+/Vaa5aTmed55jc/h5n3fuaZmWqtBQAAejBr1AMAAIDxEq8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAJdqqqDqupbVXVrVT1/ktv6QFWdcA/Xv6mqTp3MbWzCmP6iqm4a3r8HTeVtA2zNyve8AlOlqv4zyUOSrEvy6ySfTvKq1tqtm7CtLyQ5r7X2ns08xqcm+VhrbbfNud0JjmFOkl8lObC1tnoD189Lcn2SOa21O8dc/uEkP2yt/c0UDRVgytnzCky157bWdkjyuCT7Jfm90KqqbcaxnYcn+fpmHtvW4iFJtssU3b8a8HoAdMGTFTASrbUfJfm3JHsnSVW1qnplVX0rybeGlx1bVd+uql9U1XlVtevw8u8keWSS/zN8W33bqjqmqr5RVbdU1Xer6s/X31ZVPbWqflhVr62qn1TVjVV1zJjrP1xVb62q7Ydj2nW43VurateqOrGqPjZm+QOr6stVtaaqVg/31q6/7iXD27+lqq6vqhdt6P4Px/zuqrpheHr38LI9kvzHcLE1VfXFTZ3jexnn/62qk6rq4iT/L8kjq+qJVXV5Vd08/N8njln+/lV12nDufjScr9nD6/6oqi4crvezqjpjU8cMcG/EKzASVfWHSZ6d5KoxFz8/yQFJ9qqqpyX5uyT/PclDk3wvyb8kSWvtUUm+n+Fe3NbaHUl+kuQ5SXZMckySd1XV48Zs+w+S3D/Jw5IsTfK+qnrA2DG11n6d5LAkNwy3u0Nr7Ya7jfthSS5I8tYkD0zyuiSfrKoHD+P3vUkOa63dL8kTk1y9kSlYluTAJAuSzE+yf5K/aa1dl+Sxw2V2aq09bWNzeE/uaZxjFluc5OVJ7pfkluHy703yoCT/kOSCMcfb/nOSO5P8UZJ9kxya5GXD696S5HNJHpBktyT/uCljBhgP8QpMtXOqak2SlUkuTPK2Mdf9XWvtF62125K8KMnprbVVwzj96yRPGB7v+Xtaaxe01r7TBi7MIKaeNGaRtUn+trW2trX26SS3Jnn0Joz/z5J8urX26dbaXa21f09yRQYhniR3Jdm7qu7TWruxtbaxt/5fNBzPT1prP03yvzKIyYn42XCv6prhnB49gXEmyYdba18fHjd7aJJvtdY+2lq7s7W2PMk3kzy3qh6SQdS/urX269baT5K8K8mfDrezNoPDOHZtrd3eWls5wfsBMG7iFZhqz2+t7dRae3hr7S+HobreD8ac3zWDva1JkuGHun6ewZ7T31NVh1XVJcNDDNZkEGk7j1nk52M/3JTBW+U7bML4H57kBXeLxoOTPHS45/aFSV6R5MaquqCqHrOR7fzO/Rue33WCY9l5OJc7tdZ2SvKJ8YxzzDIbne8xY3rYcFtzhvdp/bZOTrLLcLnXJ6kkl1XV16vqpRO8HwDjNp4PRQBMlbFff3JDBtGUJBm+Jf+gJD+6+0pVtW2STyZ5cZJzW2trq+qcDIJqMmPYkB8k+Whr7dgNrtzaZ5N8tqruk8Fb9qfkd/cAr7f+/q3fM7v78LLN5R7HuX64GxjPWLsn+cxwW3dkEMt33m2ZtNZ+nOTYJKmqg5N8vqouaq19exLjB9gge16BrdUnkhxTVQuGcfq2JJe21v5zA8vOTbJtkp8mubOqDsvgbfBNcVOSB1XV/Tdy/ccyeCv9mVU1u6q2G34gbLeqekhVPW8Y2ndkcGjCuo1sZ3mSvxkeK7tzkjcPt725bHScG1n+00n2qKqjq2qbqnphkr2SnN9auzGDwzDeWVU7VtWsqnpUVT0lSarqBWO2+8sMonhj9xtgUsQrsFVqrX0hyQkZ7FG9Mcmj8l/HWN592VuSHJfkXzOIp6OTnLeJt/vNDMLyu8O3yHe92/U/SHJEkjdlEMs/SPI/M3g+nZXktRnsxfxFkqck+cuN3NRbMzgG9ZokX02yanjZZnEv49zQ8j/P4ANvr83g8IzXJ3lOa+1nw0VenMEfCddmMMdn5b8OQdgvyaVVdWsG8/5XrbXrN9d9ARjLjxQAANANe14BAOiGeAUAoBviFQCAbohXAAC6IV4BAOjGhH6kYOedd27z5s3bQkMBAIDkyiuv/Flr7cEbum5C8Tpv3rxcccUVm2dUAACwAVV195+r/i2HDQAA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADdEK8AAHRDvAIA0A3xCgBAN8QrAADd2GbUA4Ce3HXXXbnkkkty5513jnookzJr1qzsv//+mTt37qiHkltvvTWrVq0a9TCy0047ZZ999hn1MAC4F+IVJuCrX/1qDj300DzucY8b9VAmZc2aNZkzZ04++tGPZq+99hrZOC666KIsWbIku+yyS7bddtuRjaO1lksvvTS/+c1vRjYGAMZHvMIErFu3LnvssUcuuuiiUQ9lUlprOfXUU/OUpzwly5Yty3HHHZdZs6buKKLbb789J5xwQj7xiU/kgx/8YA4//PApu+0NWbdu3VaxFxqAe+eYV5iBqirHHntsLrnkkpx11lk55JBD8v3vf39Kbnv16tXZb7/9cv3112f16tUjD1cA+iJeYQZ71KMelQsvvDCHHnpoFi5cmI985CNprW2R21q3bl3e/va35xnPeEZe//rX58wzz8zOO++8RW4LgOnLYQMww82ePTtvfOMb86xnPSuLFy/Oueeem5NPPnmzhuV3vvOdLFmyJHPnzs0VV1yR3XfffbNtG4CZxZ5XIEmyYMGCXH755XnkIx+Z+fPn54ILLpj0NltrOeWUU3LggQfmyCOPzOc//3nhCsCk2PMK/NZ2222Xd7zjHXnuc5+bJUuW5Lzzzss73/nO7LDDDhPe1o9//OMce+yxueGGG3LhhReO9FsNAJg+7HkFfs+Tn/zkrF69OmvXrs38+fNz8cUXT2j9s88+OwsWLMiCBQvyla98RbgCsNnY8wps0I477pjTTz895557bo488sgcc8wxOfHEE+/xK6VuvvnmHHfccfnyl7+cc845JwceeOAUjhiAmcCeV+AeHXHEEVm9enWuvfbaHHDAAfna1762weVWrFiR+fPnZ/vtt8/VV18tXAHYIsQrcK922WWXfOpTn8pxxx2XRYsW5Z3vfGfWrVuXJLntttvymte8JosXL84HPvCBvP/978/2228/4hEDMF2JV2BcqirHHHNMLrvsspx33nl52tOelvPPPz8LFy7Mj370o6xevTrPetazRj1MAKa5Lo55raot9sXpwMQ84hGPyBe/+MW8613vynOueFF+teyfctRRR6WqRj00mFG8NjJT2fMKTNjs2bPzute9Lkly9NFHC1cApox4BQCgG+IVAIBuiFcAALohXgEA6IZ4BQCgG+IVAIBuiFcAALohXgEA6IZ4BYAZbPny5dl7770ze/bs7L333lm+fPmoh8RWYGt+XHTx87AAwOa3fPnyLFu2LKeddloOPvjgrFy5MkuXLk2SHHXUUSMeHaOytT8u7HkFgBnqpJNOymmnnZZFixZlzpw5WbRoUU477bScdNJJox4aI7S1Py6qtXbPC1S9PMnLk2T33Xd//Pe+972pGNfdxzDltwn35N7+u5kxTrx/cuLNox7FpK1bty7bbOONKPoz2eei2bNn5/bbb8+cOXN+e9natWuz3XbbZd26dZMdHp3aGh4XVXVla23hhq6712fr1toHk3wwSRYuXDiyV2yxwNZg1apVednLXjbqYbAFzJo1y4s1XdkcO3b23HPPrFy5MosWLfrtZStXrsyee+456W3Tr639ceGwAQCYoZYtW5alS5dmxYoVWbt2bVasWJGlS5dm2bJlox4aI7S1Py68TwYAM9T6D9+86lWvyje+8Y3sueeeOemkk7aKD+UwOlv740K8AsAMdtRRR201UcLWY2t+XDhsAACAbohXAAC6IV4BAOiGeAUAoBviFQCAbohXAAC6IV6BTXLjjTcmSW655ZYRjwSAmUS8AhN21llnZd99902SzJ8/PytXrhzxiACYKbqI19baqIcAJFmzZk0WL16cN73pTTn33HOTE2/Ou9/97rzgBS/IG97whtxxxx2jHiLMGF4bmam6iFdg9L7whS9kn332yY477pirrroqBxxwQJLkec97XlavXp3rrrsu+++/f6655poRjxSA6Uy8Avfotttuy6tf/eosWbIkp5xySt73vvdl++23/51ldtlll5x99tk5/vjj8/SnPz1///d/n3Xr1o1oxABMZ+IV2Kgrr7wyj3/843PTTTflmmuuyTOf+cyNLltVeclLXpLLL788F1xwQRYtWpTrr79+CkcLwEwgXoHfc+edd+Ytb3lLDjvssJxwwglZvnx5HvjAB45r3Xnz5mXFihU54ogjsv/+++f00093bB4Am414BX7Hddddl4MPPjhf+tKXsmrVqhx11FET3sasWbPy2te+NitWrMh73/vePP/5z89NN920BUYLwEwjXoEkg08uv//9789BBx2UxYsX5zOf+Ux22223SW1z7733zmWXXZbHPvaxWbBgQc4555zNM1gAZqxtRj0AYPRuuOGGvPSlL80vfvGLrFy5Mo9+9KM327bnzp2bt73tbTn88MOzZMmSnHvuuXnPe96THXfccbPdBgAzhz2vMMOdccYZ2XffffOEJzwhF1988WYN17EOOuigXH311Zk7d27mz5+fCy+8cIvcDgDTmz2vMEP98pe/zCtf+cqsWrUq559/fvbbb78tfps77LBDTj755FxwwQU5+uijc/TRR+ctb3lLtttuuy1+2wBMD+IVJmDNmjW56qqr8qlPfWrUQ5mUNWvW5M1vfnP+5E/+JKtWrcp973vfKb39ww8/PKtXr84rXvGK7Lffflm2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
"
>
</div>

</div>

</div>

</div>

</div><div class="jp-Cell jp-CodeCell jp-Notebook-cell jp-mod-noOutputs  ">
<div class="jp-Cell-inputWrapper">
<div class="jp-Collapser jp-InputCollapser jp-Cell-inputCollapser">
</div>
<div class="jp-InputArea jp-Cell-inputArea">
<div class="jp-InputPrompt jp-InputArea-prompt">In&nbsp;[&nbsp;]:</div>
<div class="jp-CodeMirrorEditor jp-Editor jp-InputArea-editor" data-type="inline">
     <div class="CodeMirror cm-s-jupyter">
<div class=" highlight hl-ipython3"><pre><span></span> 
</pre></div>

     </div>
</div>
</div>
</div>

</div>