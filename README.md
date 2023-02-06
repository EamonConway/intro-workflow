#Transmission model

To model the transmission of COVID-19 throughout the population of interest we have extended a previous individual-based model \cite{Sheareretal} to account for loss of protection over time since vaccination. Waning of immunity is accounted for by explicitly modelling the boosting and decay of each individual's neutralising antibody titre. This approach overcomes limitations of the typical SIRS model, where waning is modelled as an exponential distribution (or more generally as a phase type distribution), and enables us to investigate the possible impacts of variants of concern using the expected fold change in neutralising antibody titre.  

Each individual within the simulation is constructed using a known age, their corresponding age bracket within the contact matrix, a list of their vaccination history and the decay rate of their neutralising antibodies. Note that each individual's entire vaccination history must be known {\it a priori} and is an input into the transmission model. This is in contrast to other IBMs that simulate the time of vaccination, however, this is a conscious design choice. By not directly simulating the vaccination roll-out within the transmission model, we have the increased benefit of being able to directly incorporate known vaccination information and can quickly and efficiently switch out different vaccination scenarios. This is an important feature of the IBM when working with policy makers, allowing for the quick adaptation of the model to help answer policy relevant questions. At the point of initialisation each individual is assumed to be susceptible and to have zero neutralising antibodies. Once an individual is constructed, they are dynamically stored within a vector. 

The spread of infection is modelled by directly simulating contact between infectious and susceptible individuals. For an infectious individual $i$ in age bracket $n$, we sample the number of contacts that they make in age bracket $m$ from a negative binomial (NB) distribution, such that,
$$
C_m \sim \text{NB}\left(r\delta t, \frac{\Lambda_{n,m}}{r + \Lambda_{n,m}}\right),
$$
where $C_m$ (contacts/timestep) is the number of contacts made this timestep by the individual in age bracket $m$, $r$ (contacts/day) is the overdispersion parameter, $\delta t$ (day) is the size of the current time step and $\Lambda_{n,m}$ (contacts/day) is the mean number of daily contacts between age bracket $n$ and $m$\footnote{Note that we have used the definition of the negative binomial distribution where $\text{NB}(r,p)$ corresponds to probability density function $f(k)={k + r -1 \choose r} p^k (1-p)^r$.}. We then sample $C_m$ contacts uniformly from individuals within the $m$th age bracket. If contact $j$ is susceptible we determine if infection occurred using,
$$
    I \sim \text{Bernoulli}(\tau_i\xi_j),
$$
 where $I$ is an indicator variable for successful infection, $\tau_i$ (dimensionless) is the infectiousness of the infectious individual $i$ and $\xi_j$ (dimensionless) is the susceptibility of the contact $j$. We note that $\tau_i$ depends upon the transmission potential of the population of interest\cite{TPpaper} and both $\tau_i$ and $\xi_j$ depend upon the underlying immunological model, which is described in \Secref{sec:ImmuneModel}, and the heterogeneous characteristics of the individual of interest (for example, $\tau_i$ will be altered depending on the symptom status of the infectious individual and both $\tau_i$ and $\xi_j$ will depend upon age). This process of generating contacts and the resulting infections is repeated over all age brackets.

At the point of infection, whether an individual will be asymptomatic or symptomatic is sampled from 
$$
    Q \sim \text{Bernoulli}(q_i),
$$
where $Q$ is an indicator variable for symptomatic infection and $q_i$ is the probability that individual $i$ will be symptomatic, which depends on the age and neutralising antibody titre of individual $i$. We also sample the time that the newly exposed individual becomes infectious, the time for the onset of symptoms and the time that the individual will recover. Given mandated isolation requirements in Australia at the time of study, we further simulate the time of isolation in relation to symptom onset, which reduces the individual's duration of infectiousness. Note that each of these times are sampled from known distributions based on local case data \cite{TTIQpaper?}. These data have also been used to estimate the likely constraining impact on transmission of active case and contact finding and management, collectively termed test, trace, isolate, quarantine (TTIQ) \cite{TTIQpaper}. We include a factor to account for a 'partial TTIQ' effect under high case loads as in earlier work \cite{thresholdspaper}.
