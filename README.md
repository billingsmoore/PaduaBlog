# Padua Conference Blog

I'm unfortunately unable to attend the Buddhism and machine translation workshop in Padua because of a family emergency, but hope to be participating as much as possible from a distance. In lieu of the out-of-session chats that I suspect will be some of the most interesting parts of the conference, I'll be putting some (unasked for) thoughts here for anyone who might (foolishly) want my opinion as things go on.

These will be rough, half-baked thoughts of the kind that might be brought up in conversation over lunch -- not well-researched, or deeply considered opinion pieces, but hopefully will be of some use to someone. I'll also put links here for things as they come up during the conference.

Here's where you can find aggregated benchmark results for Tibetan-language AI applications: https://huggingface.co/spaces/billingsmoore/tibetan-leaderboard

Reflections are in reverse-chronological order so the most recent sessions are at the top. Linked slides are in either .pptx or .html format. HTML can be opened in our browser for viewing.

## Reflections on Day 2 - Agents, Evaluations, Archives

I was luckily able to attend a couple of the unconference sessions today. The first was focused on agentic AI. We got to see what Greg Forgues from Tsadra has been working on, and I presented the agentic work that we’ve been doing at the Khyentse Vision Project. In the second, I presented some of the work that I’ve been doing for the Garchen Archive to archive Garchen Rinpoche’s teachings from video and audio format, and then covered some of the translation quality review work being done at KVP.

In this post, I will recap (with some editorializing) both conversations out of chronological order because I think there is enough overlap that it makes more sense to organize things into a more coherent arc. My goal is to give an overview of the sessions for those who weren’t in attendance, and to extrapolate what lessons might be available here.

### Agentic Systems

The agentic session brought to light some really interesting work, and I think the differences in what’s being done reflect some of the differences in workflow across organizations, and the corpora that they are working with.

Greg’s approach relies on a pretty large number of agents and sub-agents. If I recall correctly, it was north of 14 or so for translation. These agents draw from a variety of data sources from S3 buckets to Neo4j graphs and then interact based on all of that. It’s a really complex system and I’m very much looking forward to a fuller write up of the details. He also invited folks to email him with any questions. I’ll omit his contact info here in case this page is crawled by spammers, but I’m sure you can grab his email address from someone.

The broad design pattern, known as a “Second Brain” is a popular one, though, and you can learn more about it online. This video presents how to set up a similar system for yourself. These are very cool systems, though their complexity can be intimidating. The complexity can be necessary, if you are working on material like Greg’s where there are regularly Tibetan, Sanskrit, and Chinese witnesses as well as commentaries to be compared and cross-referenced. 

On the KVP side, I presented on our Translation and Editor agents and gave a bit of a deep-dive on their design and the quality benefits of particular tools and prompt designs in those agents. [You can find the Translation Agent slides from that presentation here.](/slides/TranslationAgentSlides.html) [And the slides for the Editor Agent are here.](/slides/EditorAgentSlides.html) One of the core drivers of that work has been staying as lean and efficient as possible. This not only reduces short term costs, but also responds to broader concerns about reliance on the large AI models produced by corporate tech companies. We are keen to ensure that anywhere that AI is appearing in our workflow its impact is validated by evidence and that each additional step brings measurable benefit.

The KVP approach is also importantly aligned with the existing translation workflow there. KVP has an in-house Translation team that produces and reviews drafts before sending them to the Editorial team for more high-level review and quality feedback. This segments the quality control process into multiple specialized steps across basic translation correctness, grammatical mechanics, stylistic quality, etc. and our approach with AI is in support of that process, rather than aiming at an end-to-end AI system.

Our Translation agent is tasked with producing a drafted translation of a segment of a text with the goal being an output that has strict fidelity to the underlying Tibetan meaning. It is equipped with a glossary, a translation memory database, and a review tool to check its work (discussed at length below).

Our Editor agent is tasked with catching high-level grammatical, formatting, and stylistic errors and providing suggested fixes. It also catches terminological issues and semantic issues and flags them for human editors. This is done in separate passes with distinct tiers, so that different types of issues can be addressed in different ways as defined by internal standards.

These agents are intended to reflect and assist the Translation  - Editorial distinction in our workflow and are designed around the needs and standards of those teams. Other organizations often commission a translation from a translator who works on their own and then submits a manuscript for review and publication. These translators and organizations may prefer a more end-to-end approach, or they may prefer to use or not use pieces of a larger pipeline. Maybe they’d like something that helps correct grammatical mistakes, but would find an AI suggesting terminological changes to be distasteful. Maybe they like the idea of having AI help polish their work, but can’t imagine allowing AI to produce a  first draft for them. 

I think this variability in workflows across our community helps to motivate a modular building strategy. Having a set of agents, small models,customized prompts and so on, that can be easily plugged into one another when desired or used as standalone tools. This provides a thorough toolkit without forcing anyone to adopt functionality they don’t want. 

This modular approach would also allow individuals and organizations to pick up individual pieces and alter them for their own workflows. Maybe one org loves everything about the pipeline but the editing stage doesn’t match their preferred tone. In a modular system they could tweak just that piece and plug it back in without a problem.

As a representative example of a modular build: one of the key elements of the Translation Agent is its Review tool. This provides the model the ability to produce a translation then evaluate the quality and if it’s not above our quality threshold it can use that feedback to provide an updated translation. 

That Review tool can be easily swapped out for any other means of conveying a measure of translation quality. The “Review tool” could be an LLM call, it could be a small locally hosted model, or it could be a human who is monitoring drafted outputs as they come through. How we arrived at the current tool was a big chunk of the second session.


### Translation Quality Evaluation

#### Automated Translation Quality Evaluation - A Quantitative View

This work gets pretty technical but I want to give a high-level overview here so that folks who are interested can get a sense of what’s going on here. The slides from the presentation are linked in the relevant sections below.

Computer scientists have been working on machine translation since before “artificial intelligence” was a recognized phrase. Accordingly, how to evaluate translation quality has been an important element of this work for quite some time. Traditionally, evaluation metrics have relied on an existing gold-standard translation to compare the machine output to. For working translators, that’s not terribly useful. After all, if our point of comparison is an existing translation that we’re already happy with, it’s not clear what we’re trying to accomplish at all.

Instead, we need a “reference-free” metric. This can be done for many languages with a metric known as COMET. COMET is a small AI model that takes in a text and its translation gives a score for how good that translation is. Unfortunately, it does not work reliably for Tibetan. 

Recently, researchers have had significant success asking LLMs to rate translation quality, and at least a couple LLMs are reasonably good with Tibetan, so that seemed like a good path forward.

At KVP we collected samples of human evaluation of translations from some of our translators and tested several evaluation methods to see which one agreed most closely with the human assessments. The winner was the LLM method using Gemini. [The slides  from this portion can be found here.](/slides/TQE-slides/slides.html)

Gemini’s outputs were then made more consistent and more reliable by optimizing the prompt, as well as altering things like the model temperature, the amount of “thinking”, the number of times we prompt the model, etc. This process brings Gemini into a level of agreement with human assessments that rivals the level agreement between human evaluators. [The slides  from this portion can be found here.](/slides/TQE-slides/GEMBAOptimizationslides.html)

Once Gemini had been dialed in, we then used Gemini to produce a larger set of example evaluations and we were able to use that as training data to finetune a much smaller model that can be run on low-cost hardware like a laptop. On a 5-point scale (“Rate this translation on a scale of 1-5”) this small model is within 1 point of human assessments 80% of the time. This is less accurate than Gemini, but also much less expensive to run. There are trade-offs in model size. [The slides  from this portion can be found here.](/slides/TQE-slides/deck.html)

#### A Broader Discussion

The quantitative work above reflects one goal of quality evaluation. This Review tool plays a particular role in the Translation Agent, but is also useful for flagging potentially problematic passages in a drafted translation so that human evaluators can get a sense of which portions of a text need the most attention.

What this tool does not do is point out when the passive voice has been used inappropriately, or when translation uses the Tibetan term Khandro where an organizational style guide suggests the Sanskrit term Dakini would be preferred (with or without diacritics?). 

Because KVP splits these types of review into multiple pieces, it is valuable to have these as separate tools, even outside of any questions about what sorts of things might be valuable to an agent. Additionally, a more high-level review focusing on terminology or style may overlook that the fundamentals of the translation have made a mistake (e.g. a negative modifier has been overlooked) and a quick (cheap) pass to make sure that there’s nothing severe being missed could be a valuable addition to that.

At KVP, though, that high level work is handled by our Editor Agent. This allows translation review in the Translation agent as well as flagging of problematic passages for review within the Translation Team, without muddying their work with concerns that are more appropriate for the Editorial Team. The Editor agent can then assist in the Editorial Teams domain of expertise in a separate phase.

For translators or organizations where this multi-phase workflow is not their usual way of doing things, collapsing this functionality into a single end-to-end approach may better serve their goals. Alternatively, using just one or the other of these two tools: the numerical scoring, or the high-level corrections; may be more in accordance with what they are looking for.

Again, this points to the value of modularity and a certain sort of pluralism in the way that we are building tools. I don’t think that it is the place of the engineer or data scientist to impose notions of translation quality, nor are we likely to arrive at a single consensus opinion on what makes a good translation and what sort of quality decisions we are comfortable handing off to an AI. Thus, we should be looking for ways to design evaluation methods that can complement one another and be used one-at-a-time or as an ensemble.

### Garchen Archive

Parallel to all of this, I also presented some of how AI is being used in the Garchen Archive. [You can see the slides from this presentation here.](/slides/ArchiveAI%20presentation.pptx)

This is exciting work to me because it’s a place where we are actively working to archive and disseminate the living tradition. This comes with some interesting challenges: working with video and audio of varying qualities; working with a speaker with a unique accent whose voice has changed over time; and working with a set of data that is actively growing as Garchen Rinpoche continues to deliver public teachings.

This work includes: automatic speech recognition to transcribe not just Garchen Rinpoche, but also his interpreters in various languages (each with their own accents); machine translation to produce translations into languages that don’t have interpreters or when live interpretation isn’t ideal for the archive; produced synthesized speech that can accurately pronounce dharma terms to make text transcripts accessible to students who might struggle to read captions or transcripts.

This is also a great case-study in rigorous quality control as each of these phases goes through human review and editing; and how that provides continuous improvement to a system. The AI drafts provide a starting point for human editors, and the human edited content becomes training data for the AI, which in term produces better drafted starting points in a virtuous cycle.


## Reflection on the Opening Session

### Can AI do Dharma?

If AI is human-like, what can it do in the way of teaching? Can we have an AI guru? This is the framing that seems to be most interesting, and controversial, for folks. But, gurus do not need to be human-like, and we may find it instructive to consider the things that are not humans but that are recognized as valid teachers.

As Michael Sheehy pointed out, the tradition (at least in places) recognizes that all appearances are the guru. Is our skepticism toward AI a failure of pure vision? 

Garchen Rinpoche offers teachings and empowerments by livestream and by recording. He often describes video recordings as being a samboghakaya of the guru. If a video can be a samboghakaya, what else might be? Where are the lines and why do they land there?

The Yuthok Nyinthik lineage considers the texts themselves to be a type of guru. Garchen Rinpoche often references a prophecy from Shakyamuni that in the future he would take the form of written letters. The prajnaparamita literature encourages practitioners to value and pay homage to the texts themselves. Perhaps AI is a possible teacher not because it is human but because it is text!

But our focus here is translation. To what degree do our concerns about teachers apply to translators? Do we expect translators to be as realized as lamas? Must they be capable of teaching or practicing all of the material that they translate? It’s only to the extent that we answer yes to those questions that we need AI to be capable of these things.

### Should AI do translation?

A translator from NYU, whose name I unfortunately did not catch, mentioned that if the goal is to have a high-quality text, it's not clear that it matters how we arrived at that text. This brings us to ask, what is the point of translation? Is it to have the text? It seems from my conversations with translators that some of them take translation to have a value in-itself. Handing this off to a machine would be a clear loss of meaningful value, then.

It's not obvious that this has been the consensus historically, though. As Andre pointed out, once the Indian corpus had been translated into Chinese, Tibetan, etc. they then carried on in their own languages, and similarly we will someday have translated the various “source language” canons into other languages and translation efforts will be primarily between non-source languages. Additionally, it is worth noting that what constitutes a “source language” for us, is often just a language that finished its translation projects before we did. 

One interesting counterpoint is the role of Sanskrit in the Vajrayana. The Vajrayana traditions have preserved Sanskrit mantras and dharanis and have historically maintained scholarly interest in the study of the Sanskrit language. What this means for translation is not clear, however, as this is an example of something that is specifically untranslated; but it seems to presage the preference in English-speaking sanghas to continue liturgical practice in their traditions' source language. Many Tibetan Buddhists recite their sadhanas etc. in Tibetan, skimming the English translation as they recite so that they can get a sense of what they are saying. If this continues to be a preference, it seems that having at least some degree of multilingualism in the sangha will be valuable in perpetuity.

### How will this affect translators?

It is crucial to remember that translation as a human activity is not an abstract thing. Translators (it will come as no surprise to many folks involved here) are people with careers, bills, families, and all of the various things that come along with being alive and in need of food and shelter.

In general, we should expect that at least some translation jobs will vanish with time. This is true independent of AI or other technologies. Eventually the Kangyur and Tengyur will be translated and the need to be fluent in Sanskrit, Tibetan, Classical Chinese, and so on, will be reduced accordingly.

AI will accelerate this process, and what serves translators and their well being is not denial of this but to respond realistically. As Lama Tharchin Rinpoche has said, you have many options when you are being chased by a tiger, but closing your eyes is not a good one. And to be clear, the machine translation researcher and the human translator are both out of a job on the day that AI translation is perfect, so I am not without ‘skin in the game’ on this.

We should, as a community, be working hard to provide technical training to translators so that they can use this technology effectively. This will empower them to not be left behind by this tectonic shift. We should also be thinking frankly about how the availability of AI impacts how we train the next generation of translators and how we will handle the shifting financial reality that this is bringing about.

### What is to be done?

In a sense, there is little in the way of bold engineering innovation to be done here. Machine translation, and natural language processing more generally, are advancing at a pace that will swiftly trivialize the progress of any individual researcher or organization in this space. 

In the market, firms compete and fail and niches get filled through trial and error. The winners get rich, the losers get laid off, and sometimes, so do the winners! We can not morally square this with Buddhist values. Nor, in this small space with limited funding, can we afford this approach. It is imperative that we are diligent in avoiding duplicated work and competition when collaboration or division of labor would be more effective and sustainable.
