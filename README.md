# Financial Indexes as Coupled Oscillators

## Objective:

The goal of this project is to apply my physics experience to finance by modeling interconnected global markets as a network of coupled oscilllators (think pendulums together on a string). By analyzing key metrics such as the resulting coupling matrix $K$, one can uncover underlying coupling strengths between markets, explore the synchronization of markets, and perform stability analysis on the system as a whole. This is not meant to predict prices, but rather probe the dynamics of the system within a given period of time.

## Introduction:

Synchronization in financial markets refers to the extent to which the behaviors of different assets or markets become correlated over time. As synchronization increases, the movements of individual stocks become more dependent on one another, reducing effective diversification. This interdependence can amplify systemic risk: a local shock to one asset or sector may propagate through the network, triggering broader market instability. For instance, if stock A is strongly coupled to stock B, a significant decline in B’s price may lead to a corresponding drop in A’s price, even in the absence of new information. High synchronization across markets may thus signal a fragile equilibrium, where localized disturbances lead to global cascades. In contrast, low synchronization implies greater robustness, as idiosyncratic shocks are more likely to remain contained. To quantify synchronization between financial markets, this project attempts to treat financial markets as a network of coupled oscillators whose phase (movement) is governed by the Kuramoto model.

## Model Overview:

The Kuramoto model is a model for the behavior of a large set of coupled oscillators, shown in equation (1).

$\dot{\theta_i} = \omega_i + \frac{1}{N}\sum_{j=1}^{N}K_{ij}sin(\theta_j - \theta_i)$ (1)

where $\dot{\theta_i}$ is the time derivative of oscillator i's phase, $\omega_i$ is the natural frequency, N is the number of oscillators, and $K_{ij}$ is the coupling matrix that describes how strongly correlated oscillator i and j are. There are assumptions in this model, including that the oscillators are identical or nearly identical and that the interactions depend sinusoidally on the phase difference between each pair of oscillators. As it stands, my Kuramoto model does not include a white noise parameter $\zeta_i(t)$ and is purely deterministic. Future tweaks and inclusions of noise parameters may be desired or necessary for better representation of the stochastic nature of financial markets.

One way of quantifying the cumulative coherence of the system of oscillators is to calculate the "order" parameters $r$ and $\psi$. They are defined as

$re^{i\psi} = \frac{1}{N}\sum_{j=1}^Ne^{i\theta_j}$ (2)

where $r$ represents the phase-coherence and $\psi$ is the average phase of the oscillators. $r$ can have values between 0 (uncoupled) and 1 (full phase-locking).

### Model Procedure:

This model can take N number of financial markets from the Yahoo! finance (yfinance python package). It has been tested on weekly closing prices for up to N = 10 major stock indexes, including ^GSPC, ^IXIC, ^FTSE, and others. Naturally, a high level of synchronization is expected among indexes, as each index aggregates the behavior of numerous underlying stocks. Testing on individual stocks would also be interesting, especially stocks from a diverse set of industries.

Since the current model is purely deterministic, I apply a third-order Savitzky–Golay filter over a 10-week window to smooth the price data for improved fitting. The objective is to analyze global synchronization and large-scale movements rather than local fluctuations, so smoothing should be appropriate as long as it does not obscure these broader trends.

After smoothing, I take the logarithm of the prices, $log(P_i(t))$, and subtract the mean across all assets at each time step, yielding $\theta_i = log(P_i(t)) - \langle log(P_i(t)) \rangle$. This normalization step mitigates differences in absolute price levels between markets or stocks, aiding numerical stability during fitting. Importantly, it does not alter the underlying Kuramoto dynamics, which are invariant under uniform (vertical) shifts across oscillators. The parameters $\omega_i$ and $K_{ij}$ are then extrapolated.

### Concerns/topics that need to be addressed:

1. For N oscillators, there are $N_p$ = N (from $\omega_i$) + N(N-1) (from $K_{ij}$ since the diagonals are zero). So the total number of parameters that need to be fit are $N_p =N^2$. So for N=10, there are 100 parameters that need fitting, which can be computationally heavy and also poses a risk for overfitting if the timespan is not long enough (not enough data). Investigation on preventing overfitting is ongoing.

2. On the topic of overfitting, how reliable are the values of $K_{ij}$? Are they actually meaningful, or did the algorithm just assign values that 'worked'? For example, are negative values meaningful?

3. The symmetry of $K$. Intuitively, it can make sense that one market can have a heavy dependence on another but not the reverse - leading to a descrepancy in $K_{ij}$ vs $K_{ji}$. But should it be somewhat symmetric or symmetric in general? Can a measure of the symmetry tell us anything? This builds off of the previous question (i.e. are the values of $K$ actually meaningful)?

4. How can I better probe synchronization for specific oscillators, not just the system as a whole (done with $r$ parameter)? The values $K_{ij}$ and $K_{ji}$ can point to how coupled they are, but is there another way that can demonstrate coupling. In my 'UnderstandingOscillators' notebook, I did some bifurcation plots, but that was with homogeneous K (not a matrix), can i do something similar if K is a matrix?
