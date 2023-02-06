Waning Immunity of COVID-19
================

# Transmission model

To model the transmission of COVID-19 throughout the population of
interest we have extended a previous individual-based model to account
for loss of protection over time since vaccination. Waning of immunity
is accounted for by explicitly modelling the boosting and decay of each
individual’s neutralising antibody titre. This approach overcomes
limitations of the typical SIRS model, where waning is modelled as an
exponential distribution (or more generally as a phase type
distribution), and enables us to investigate the possible impacts of
variants of concern using the expected fold change in neutralising
antibody titre.

Each individual within the simulation is constructed using a known age,
their corresponding age bracket within the contact matrix, a list of
their vaccination history and the decay rate of their neutralising
antibodies. Note that each individual’s entire vaccination history must
be known {} and is an input into the transmission model. This is in
contrast to other IBMs that simulate the time of vaccination, however,
this is a conscious design choice. By not directly simulating the
vaccination roll-out within the transmission model, we have the
increased benefit of being able to directly incorporate known
vaccination information and can quickly and efficiently switch out
different vaccination scenarios. This is an important feature of the IBM
when working with policy makers, allowing for the quick adaptation of
the model to help answer policy relevant questions. At the point of
initialisation each individual is assumed to be susceptible and to have
zero neutralising antibodies. Once an individual is constructed, they
are dynamically stored within a vector.

The spread of infection is modelled by directly simulating contact
between infectious and susceptible individuals. For an infectious
individual $i$ in age bracket $n$, we sample the number of contacts that
they make in age bracket $m$ from a negative binomial (NB) distribution,
such that,
$$C_m \sim \text{NB}\left(r\delta t, \frac{\Lambda_{n,m}}{r + \Lambda_{n,m}}\right),$$
where $C_m$ (contacts/timestep) is the number of contacts made this
timestep by the individual in age bracket $m$, $r$ (contacts/day) is the
overdispersion parameter, $\delta t$ (day) is the size of the current
time step and $\Lambda_{n,m}$ (contacts/day) is the mean number of daily
contacts between age bracket $n$ and $m$. We then sample $C_m$ contacts
uniformly from individuals within the $m$th age bracket. If contact $j$
is susceptible we determine if infection occurred using,
$$I \sim \text{Bernoulli}(\tau_i\xi_j),$$ where $I$ is an indicator
variable for successful infection, $\tau_i$ (dimensionless) is the
infectiousness of the infectious individual $i$ and $\xi_j$
(dimensionless) is the susceptibility of the contact $j$. We note that
$\tau_i$ depends upon the transmission potential of the population of
interest and both $\tau_i$ and $\xi_j$ depend upon the underlying
immunological model, which is described in , and the heterogeneous
characteristics of the individual of interest (for example, $\tau_i$
will be altered depending on the symptom status of the infectious
individual and both $\tau_i$ and $\xi_j$ will depend upon age). This
process of generating contacts and the resulting infections is repeated
over all age brackets.

At the point of infection, whether an individual will be asymptomatic or
symptomatic is sampled from $$Q \sim \text{Bernoulli}(q_i),$$ where $Q$
is an indicator variable for symptomatic infection and $q_i$ is the
probability that individual $i$ will be symptomatic, which depends on
the age and neutralising antibody titre of individual $i$. We also
sample the time that the newly exposed individual becomes infectious,
the time for the onset of symptoms and the time that the individual will
recover. Given mandated isolation requirements in Australia at the time
of study, we further simulate the time of isolation in relation to
symptom onset, which reduces the individual’s duration of
infectiousness. Note that each of these times are sampled from known
distributions based on local case data . These data have also been used
to estimate the likely constraining impact on transmission of active
case and contact finding and management, collectively termed test,
trace, isolate, quarantine (TTIQ) . We include a factor to account for a
‘partial TTIQ’ effect under high case loads as in earlier work .

When the infectious individual recovers, we store the individual’s age,
time of symptom onset, neutralising antibody titre at exposure, symptom
indicator ( $Q$ ), the number of individuals they infected, their vaccine
status (what was the latest vaccine they received) at exposure, the time
they were isolated from the community and the number of times that they
have been infected. This generates a line list of infections that is
used to model clinical outcomes ().

## Immunological model

Within both the transmission model ([Section 1](#sec-TransmissionModel))
and the clinical outcomes model (**?@sec-ClinicalOutcomes**) we include
an immunological response to COVID-19. This immunological response is
handled by directly modelling each individual’s neutralising antibody
titre. By using the model of and we can relate an individual’s
neutralising antibody titre to their protection against all outcomes of
interest: infection, symptomatic disease, onward transmission given
breakthrough infection, hospitalisation and death. The fact that each
individual is assigned their own neutralising antibody titre results in
inter-individual variation that leads to varying distributions of
protection throughout the community, impacting the resulting dynamics.
In the Australian context, by the time Delta and Omicron outbreaks
occurred almost all immunity was vaccine-derived, therefore the model is
initialised by considering neutralisating antibody titre from the
vaccine roll out alone.

An individual’s neutralising antibody titre can be increased by a
variety of exposures. The processes that we consider are: the first,
second and booster dose of a vaccine, or infection occurring in an
unvaccinated or vaccinated individual. Note that for simplicity we
assume that infection prior to or following vaccination results in the
same titre of neutralising antibody. Immune responses are stratified by
the type of vaccine product, AstraZeneca (AZ) or mRNA vaccine (Pfizer or
Moderna), that the individual has received based on supply and
distribution data.

At the time of a boosting process, we sample the level of neutralising
antibody titre that is acquired from,
$$\log_{10}(a_i^0) \sim \mathcal{N}(\mu^x_j, \sigma^2),$$ where $a_i^0$
is the neutralising antibody titre that individual $i$ is boosted to,
$\mu^x_j$ is the mean neutralising antibody titre against strain $x$ of
the population after boosting process $j$ and $\sigma^2$ is the variance
of neutralising antibodies across the population.

The mean neutralising antibody titre, $\mu^x_j$, is set using logical
rules based upon the infection and vaccination history of the
individual. In our work it is assumed that an unvaccinated individual
will have an average neutralising antibody titre of 0.0 on the
$\log_{10}$-scale after exposure. This is our baseline measurement and
is used to calibrate across multiple neutralising antibody studies. For
an individual that has no prior exposure to COVID-19, their vaccination
induces an antibody response such that, <span
id="eq-meanneuts">$$\mu^x_j = \mu_j^0 + \log_{10}(f_x), \qquad(1)$$</span>
where $\mu_j^0$ is the mean level of neutralising antibody titre for
vaccine $j$ against a base strain of COVID-19 (for us this is Delta) and
$f_x$ is the fold change in neutralising antibody titre between the base
strain and strain $x$. To account for the effect of exposure to COVID-19
prior or post vaccination, we use an altered form of . For brevity, we
have used to list the equations used to obtain $\mu^x_j$ with infection
rather than include a description within text.

It is assumed that an individual’s titre of neutralising antibodies will
decay after boosting. This decay is assumed to be exponential,
therefore, where $a_i$ is the time dependent neutralising antibody titre
of individual $i$, $k_a$ is the decay rate of neutralising antibodies
and $t$ is the time from the last boosting process (to limit the
computational cost of constantly converting neutralising antibodies, all
equations are expressed in terms of $\log_{10}(a_i)$ in our work).

To convert the neutralising antibody titre of an individual to their
protection against any disease outcome, $\rho_\alpha$, we use where $k$
is governs the steepness of the logistic curve (logistic growth rate),
and $c_{\alpha}$ defines the midpoint of the logistic function for
disease outcome $\alpha$.

The immunological model interacts with the transmission model by
altering the probability that an individual develops symptoms, $q_i$,
their rate of onward transmission given breakthrough infection,
$\tau_i$, and the contact’s level of susceptibility, $\xi_j$.

The susceptibility of contact $j$ is, where $\rho_\xi$ is the protection
against infection and $\xi_i^0$ is the susceptibility of the $i$th
individual if they were completely COVID naive. The probability that
individual $i$ develops symptoms is governed by, where $\rho_q$ is the
protection against symptomatic infection and $q_i^0$ is the probability
of symptomatic infection for individual $i$ if they were completely
COVID naive (zero neutralising antibody titre).

To model the onward transmission rate more care must be taken. It is
assumed in our model that asymptomatic individuals are 50% less likely
to infect their contacts when compared to their symptomatic counterpart.
However, this reduction in transmission due to asymptomatic infections
is not accounted for in the clinical trial data used to calibrate the
protection against onward transmission. To avoid double counting the
effect of the neutralising antibodies we alter the functional form for
the rate of onward transmission to, where $s$ is either 0.5 or 1
depending upon whether the individual is asymptomatic or symptomatic
respectively, $\rho_\tau$ is the protection against onward transmission
and $\beta_i$ is the baseline (zero neutralising antibody titre)
infectiousness of the infector. Note that $\beta_i$ depends upon the age
of the individual and the expected transmission potential of the
population (transmission potential does not vary in time for our
scenarios). For a full derivation of and , see .

% \begin{itemize} %
% \end{itemize}

To reduce the computational cost of updating the immunological component
of the transmission model for each individual at every timestep, we only
solve the immunological component of the IBM when we require the
protection against an outcome of interest. This is done by storing the
time of last boost of neutralising antibodies and the titre that the
individual was boosted to. When required, we update the individual’s
neutralising antibody titre to the current timestep using this stored
information.

The clinical outcomes model uses the transmission model as an
intermediary between the immunological response of each infected
individual and their corresponding clinical outcome. This is done by
outputting each infected individual’s neutralising antibody titre at the
point of exposure, a symptom indicator and their time of symptom onset
for use within the clinical pathways model.

The immunological model determines the probability of hospitalisation,
ICU requirement and death based on observed relationships between
neutralising antibody titres and clinical endpoint outcomes from
efficacy studies. For a symptomatic individual $i$, the probability of
hospitalisation is given by where $p^0_{H|E}$ is the baseline
probability of hospitalisation given infection, $\rho_h(a_i)$ is the
protection against hospitalisation, $a_i$ is individual $i$’s
neutralising antibody titre at the point of exposure, and is the
function that uses odds ratio $r$ and baseline probability $p$ to
compute an adjusted probability.

If individual $i$ is hospitalised, the probabilities governing which
hospital pathway is chosen are altered such that, and, where
$p^0_{\text{ICU}| E}$ is the baseline probability of requiring the ICU
given infection, $p_{H_D|E}^0$ is the probability of death on ward
(without visiting ICU) given infection and $\rho_\text{D}(a_i)$ is the
protection against death given infection.

If individual $i$ is in the ICU, then their probabilities of death in
the ICU, $p_{\text{ICU}_D|\text{ICU}}^i$, and death on the ward given
they left ICU without dying, $p_{W_D|\text{ICU}_D^c}^i$, are altered
such that, and, where $p^0_{\text{ICU}_D|E}$ is the baseline probability
of dying in the ICU and $p_{W_D|E}^0$ is the baseline probability of
dying in the ward after returning from the ICU. Note that we assume no
difference between the protection from hospitalisation given infection
and the protection from ICU given infection here.

%
To determine all parameters in and , we use a re-implementation of and
in a Bayesian framework. This allows us to calibrate the level of
protection, which is analogous to vaccine efficacy for individuals with
no exposure to COVID-19, to observed clinical data. The model fit in
takes in a range of data relating neutralising antibody levels to
efficacy, and estimates of vaccine efficacies from a range of studies to
estimate efficacy over time against the Delta variant. To estimate the
efficacies against the Omicron variant, Golding and colleagues estimate
an ‘escape’ parameter for the Omicron variant relative to the Delta
variant. This was estimated using estimates, the relative rates of
infection in Danish households between Omicron and Delta to estimate the
the relative $R_0$ between the variants, and early evidence of vaccine
efficacies against Omicron from the UK to understand the level of
vaccine escape. This was then combined with the information fit on the
Delta variant to model waning over time for both the Delta and Omicron
variants.

% \begin{itemize} %
% \end{itemize}

Presented in and are the median levels of protection against all
outcomes of interest for both the Delta and Omicron strain. The
parameters used to generate these results are listed in . Here we can
see the decay of protection as time progresses. Notably, the worst
protection results from vaccination with AZ, which was the recommended
vaccine for older individuals in Australia. A further complicating
factor is that individuals that received AZ were prioritised early in
the vaccination program, meaning that their immunity has likely waned
significantly compared to the younger individuals who received their
dose in late 2021.

{#sec-ClinicalOutcomes}

The clinical outcomes model is based on the clinical model from and is
an extension of the work done in . This model extends on previous work
by restructuring the existing model as a continuous-time, stochastic,
IBM, where the neutralising antibody titre and age of the individual
determines their transition probabilities. Furthermore, an emergency
department (ED) compartment has been added as noted that limited ED
consult capacity can cause a bottleneck that prevents admission to
hospital. The full compartmental structure of the clinical outcomes
model is depicted in .

All parameters governing the pathway each individual takes through the
health system are altered depending upon their individual neutralising
antibody titre. The evaluation of these transition probabilities are
explained in full in and baseline parameters associated with Delta
severity in an unvaccinated individuals are given in .

For symptomatic individual $i$ in the line-list outputted from the
transmission model, we determine if they are hospitalised by sampling,
where $H$ is an indicator variable for hospitalisation and $p_{H| I}^i$
is the probability that individual $i$ is hospitalised given symptomatic
infection. If individual $i$ requires hospitalisation they will present
to the ED, where they may not be seen due to capacity limitations. ED
consult capacity is modelled by admitting only the first $C_{ED}$
presentations to ED each day. If individual $i$ is not seen, they will
present again to the ED with probability $1 - p_{\text{L}|\text{ED}}$
after $\tau_{\text{L}|\text{ED}}$ days sampled from, where
$p_{\text{L}|\text{ED}}$ is the probability that an individual does not
present again to the ED and $\kappa_{\text{L}|\text{ED}}$ and
$\theta_{\text{L}|\text{ED}}$ are the shape and rate parameters of the
gamma distribution respectively. For individuals that do not return to
ED and are therefore not admitted to hospital, their age, neutralisation
titre upon exposure and number of presentations to ED are recorded such
that these can be used to understand possible excess mortality due to ED
capacity limits. If individual $i$ is admitted to hospital we determine
what hospital pathway they will follow.

There are three initial pathways for hospitalised individual $i$.
Individual $i$ will either recover and be discharged from a ward bed,
die in a ward bed, or move to an ICU bed; as the three pathways have
different length of stay distributions they modelled as three separate
compartments $H_R$, $H_D$ and $\text{ICU}_{pre}$. To determine which
pathway individual $i$ will follow, we sample from, where $X_h$ is the
sampled hospital pathway, is a vector containing the probability of
transitioning into $\text{ICU}_{pre}$, $H_D$, or $H_R$ respectively,
$p_{\text{ICU}|H}^i$ is the probability that individual $i$ is admitted
to ICU given they are hospitalised and $p_{H_D|\text{ICU}^c}^i$ is the
probability that individual $i$ dies on ward given that they are in
hospitalised and are not going to ICU. If individual $i$ requires the
ICU, they follow a further ICU pathway to determine their final outcome.

The pathway through the ICU also consists of three different components.
Within the ICU pathway an individual will either die in the ICU
($\text{ICU}_D$), die in a ward bed after leaving the ICU
($\text{ICU}_{WD}$), or recover and be discharged from a ward bed after
leaving the ICU ($\text{ICU}_{WR}$). We sample which pathway is taken
within the ICU from, where $X_\text{ICU}$ is the sampled ICU pathway, is
a vector containing the probability of transitioning into the
$\text{ICU}_D$, $\text{ICU}_{WD}$ or $\text{ICU}_{WR}$ compartment
respectively, $p_{\text{ICU}_D|\text{ICU}}^i$ is the probability that
individual $i$ dies in the ICU given they were admitted to ICU and
$p_{W_D|\text{ICU}_D^c}^i$ is the probability that individual $i$ of
dies in a ward bed after leaving ICU without dying. For an individual
that transitions into $\text{ICU}_{WR}$ or $\text{ICU}_{WD}$, they will
move into a further ward compartment, $W_R$ or $W_D$, where they will
either recover or die respectively.

Finally, the length of stay for individual $i$ in each compartment is
sampled such that, where $\tau_c$ is the time spent in compartment $c$,
and $\kappa_c$ and $\theta_c$ are the shape and rate parameter for
compartment $c$ respectively. Uncertainty is incorporated by sampling
rate and shape parameters from the posterior estimated for the
Australian Delta wave . The mean lengths of stay in each compartment by
age are given in Table .

By generating a clinical timeline for every symptomatic individual, we
can calculate hospital admissions, ICU occupancy, ward occupancy and
deaths by age in continuous-time. Furthermore, by explicitly
incorporating the effects of neutralising antibodies on protection
against each outcome, we are able to account for individual level immune
responses. Note that we assume that neutralising antibody titre levels
do not change the distribution of time spent in any compartment.
