clear; clc; close all;

% Verifica y cierra puertos seriales abiertos
if ~isempty(serialportfind)
    fclose(serialportfind);
    delete(serialportfind);
end

comPort = 'COM#';      % Puerto de entrada
baudRate = 115200;     % Velocidad de comunicación
fs_hz = ###;           % Frecuencia de muestreo en Arduino

% Configuración de la captura
tiempoDeMuestreo = 5;  
numPuntos = tiempoDeMuestreo * fs_hz;

% Conexión con Arduino
try
    disp('Estableciendo conexión con Arduino...');
    s = serialport(comPort, baudRate);
    configureTerminator(s, "LF"); 
    disp('¡Conexión exitosa!');
catch e
    disp(['Error: No se pudo conectar al puerto ' comPort]);
    return;
end

% Inicialización del Arduino
disp('Esperando inicialización (4 segundos)...');
pause(4.5);
header = readline(s);
disp(['Encabezado recibido: ' header]);

% Reserva memoria para datos
dataMatrix = zeros(numPuntos, 4);  
tiempo = (0:numPuntos-1) / fs_hz;  

% Captura de datos
disp(['Capturando datos por ' num2str(tiempoDeMuestreo) ' segundos...']);
tic;
for i = 1:numPuntos
    rawData = readline(s);
    parsedData = str2double(strsplit(rawData, ','));
    
    if numel(parsedData) == 4
        dataMatrix(i, :) = parsedData;
    elseif i > 1
        dataMatrix(i, :) = dataMatrix(i-1, :);
    end
end
disp(['Captura finalizada en ' num2str(toc, '%.2f') ' segundos.']);

% Graficación
figure;
plot(tiempo, dataMatrix, 'LineWidth', 1.5);
title('Gráfica de los Primeros 5 Segundos de Datos');
xlabel('Tiempo (s)'); ylabel('Voltaje (V)');
grid on;
legend('Salida (y)', 'Control (u)', 'Error (e)', 'Referencia (r)', 'Location', 'best');

% Limpieza
clear s;
