%% Part A: Echo Path Impulse and Frequency Response
load('path.mat'); % Load the impulse response sequence
figure;
stem(path); % Plot the impulse response
xlabel('Sample');
ylabel('Amplitude');
title('Echo Path Impulse Response');

figure;
freqz(path); % Plot the frequency response
title('Echo Path Frequency Response');

%% Part B: Composite Source Signal and PSD
load('css.mat'); % Load the composite source signal
figure;
subplot(2, 1, 1);
plot(css); % Plot the composite source signal
xlabel('Sample');
ylabel('Amplitude');
title('Composite Source Signal');

subplot(2, 1, 2);
pwelch(css); % Plot the Power Spectral Density (PSD)
title('Composite Source Signal PSD');

%% Part C: Echo Signal, Input/Output Power, and ERL
block = repmat(css, 5, 1); % Concatenate 5 blocks of the composite source signal
echoSignal = filter(path, 1, block); % Apply the echo path to the concatenated signal

figure;
plot(echoSignal); % Plot the resulting echo signal
xlabel('Sample');
ylabel('Amplitude');
title('Echo Signal');

N = length(block); % Length of the concatenated signal
inputPower = 10 * log10(sum(block.^2) / N); % Input power in dB
outputPower = 10 * log10(sum(echoSignal.^2) / N); % Output power in dB

ERL = inputPower - outputPower; % Echo-return-loss (ERL) in dB

fprintf('Input Power: %.2f dB\n', inputPower);
fprintf('Output Power: %.2f dB\n', outputPower);
fprintf('ERL: %.2f dB\n', ERL);

%% Part D: Adaptive Line Echo Cancellation
farEndSignal = repmat(css, 10, 1); % Use 10 blocks of the composite source signal as far-end signal

echoSignal = filter(path, 1, farEndSignal); % Generate echo signal using the echo path

adaptFilterLength = 128; % Length of adaptive filter
mu = 0.25; % Step size factor

adaptFilter = zeros(adaptFilterLength, 1); % Initialize adaptive filter weights
errorSignal = zeros(size(echoSignal)); % Initialize error signal

for n = 1:length(echoSignal)
    farEndSample = farEndSignal(n);
    echoSample = echoSignal(n);
    
    y = adaptFilter' * farEndSample; % Output of adaptive filter
    error = echoSample - y; % Error signal
    
    adaptFilter = adaptFilter + (mu / norm(farEndSample)^2) * conj(farEndSample) * error; % Update filter weights
    
    errorSignal(n) = error; % Store error signal
end

figure;
subplot(3, 1, 1);
plot(farEndSignal); % Plot the far-end signal
xlabel('Sample');
ylabel('Amplitude');
title('Far-End Signal');

subplot(3, 1, 2);
plot(echoSignal); % Plot the echo signal
xlabel('Sample');
ylabel('Amplitude');
title('Echo Signal');

subplot(3, 1, 3);
plot(errorSignal); % Plot the error signal
xlabel('Sample');
ylabel('Amplitude');
title('Error Signal');

%% Part E: Estimated FIR Channel Response
figure;
subplot(2, 1, 1);
freqz(path); % Plot the original FIR channel response
title('Original FIR Channel Response');

subplot(2, 1, 2);
freqz(adaptFilter); % Plot the estimated FIR channel response
title('Estimated FIR Channel Response');

%% Part F: Alternative Adaptive Algorithm (Example: LMS)
% Implement your alternative adaptive algorithm and compare it to NLMS

% Note: The code for the alternative adaptive algorithm is not provided here as it depends on the specific algorithm chosen.

