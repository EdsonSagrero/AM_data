%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%% Codigo de Matlab para la trasferencia de datos de Arduino %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

% --- 1. CONFIGURACIÓN Y PARÁMETROS ---
clear; clc; close all;

if ~isempty(serialportfind)
     fclose(serialportfind);
     delete(serialportfind);
end

% Parámetros de la comunicación y del sistema
comPort = 'COM6';          %(modificar) Puerto serial del Arduino
baudRate = 115200;         %(modificar) Velocidad, debe coincidir con el Arduino
fs_hz = 1;                 %(modificar) Frecuencia de muestreo del Arduino

% Parámetros del experimento
tiempoDeMuestreo = 5;      
porcentajeUltimosDatos = 20; 
minPuntosParaPromedio = 5; %(modificar) Mínimo de puntos para un promedio confiable

% --- 2. ADQUISICIÓN DE DATOS ---
numPuntos = tiempoDeMuestreo * fs_hz;
tiempo = (0:numPuntos-1) / fs_hz; 
dataMatrix = zeros(numPuntos, 4); 

try
    s = serialport(comPort, baudRate);
    configureTerminator(s, "LF"); 
catch e
    error(['No se pudo conectar al puerto ' comPort '. Verifique la conexión.']);
end

pause(4.5); 

for i = 1:numPuntos
    rawData = readline(s);
    parsedData = str2double(strsplit(rawData, ','));
    if numel(parsedData) == 4
        dataMatrix(i, :) = parsedData;
    elseif i > 1
        dataMatrix(i, :) = dataMatrix(i-1, :);
    end
end
clear s; 


% --- 3. ANÁLISIS Y GRAFICACIÓN ---
% Calcular puntos a promediar basado en el porcentaje
puntosCalculadosPorPorcentaje = floor(numPuntos * (porcentajeUltimosDatos / 100));


puntosParaPromedio = max(puntosCalculadosPorPorcentaje, minPuntosParaPromedio);


puntosParaPromedio = min(puntosParaPromedio, numPuntos);

% Calcular el promedio
inicioRango = numPuntos - puntosParaPromedio + 1;
datosErrorEstacionario = dataMatrix(inicioRango:end, 3);
errorEstacionarioPromedio = mean(datosErrorEstacionario);


% Creación de la gráfica
figure;
hold on;
plot(tiempo, dataMatrix(:, 1), 'r', 'LineWidth', 1.5); 
plot(tiempo, dataMatrix(:, 2), 'k', 'LineWidth', 1.5); 
plot(tiempo, dataMatrix(:, 3), 'b', 'LineWidth', 1.5); 
plot(tiempo, dataMatrix(:, 4), 'g', 'LineWidth', 1.5); 
hold off;


% Configuración de la apariencia
tituloPrincipal = 'Comportamiento entrada-salida del sistema';
title(tituloPrincipal); 
xlabel('Tiempo [s]');
ylabel('Voltaje [V]');
grid on;
xticks(0:0.5:5);
xlim([0 5]);
ylim([0 3]);
legend('Salida, y(t)', 'Control, u(t)', 'Error, e(t)', 'Referencia, r(t)', 'Location', 'northeast');

% Añadir el valor del error como texto en la gráfica
textoError = sprintf('e_{ss} = %.4f V', errorEstacionarioPromedio);
text(0.5, 2.5, textoError, 'FontSize', 10);
