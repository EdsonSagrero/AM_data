%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%% Codigo de Matlab para la trasferencia de datos de Arduino %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

clear; clc; close all;

% --- 1. Limpieza de puertos previos ---
if ~isempty(serialportfind)
    fclose(serialportfind);
    delete(serialportfind);
end

% --- 2. Configuración del Puerto Serial ---
comPort = 'COM6';       % Puerto a utilizar (Modificar)
baudRate = 115200;      % Velocidad de comunicación
fs_hz = 100;            % Frecuencia de muestreo definida en Arduino (Modificar)

% Parámetros de captura de datos
tiempoDeMuestreo = 5;   % Duración de adquisición (segundos)
numPuntos = tiempoDeMuestreo * fs_hz;

% --- 3. Establecer Conexión con Arduino ---
try
    disp('Estableciendo conexión con Arduino...');
    s = serialport(comPort, baudRate);
    configureTerminator(s, "LF"); 
    disp('¡Conexión exitosa!');
catch e
    disp('Error: No se pudo conectar al puerto.');
    disp(['Verifique que el Arduino está conectado y el puerto ' comPort ' es correcto.']);
    disp(['Error original: ' e.message]);
    return;
end

% --- 4. Preparación para la Captura ---
disp('Esperando la inicialización del Arduino (4 segundos)...');
pause(4.5);

% Lee y muestra la línea de encabezado
header = readline(s);
disp(['Encabezado recibido: ' header]);

% Pre-asignación de memoria para los datos
dataMatrix = zeros(numPuntos, 4); 
tiempo = (0.5:0.5:5)'; 

% --- 5. Captura de Datos ---
disp(['Iniciando la captura de datos durante ' num2str(tiempoDeMuestreo) ' segundos...']);
tic; 

for i = 1:numPuntos
    rawData = readline(s);
    parsedData = str2double(strsplit(rawData, ','));

    if numel(parsedData) == 4
        dataMatrix(i, :) = parsedData;
    else
        if i > 1
            dataMatrix(i, :) = dataMatrix(i-1, :);
        end
        disp(['Advertencia: Se recibió una línea de datos incompleta en la muestra ' num2str(i)]);
    end
end

tiempoTranscurrido = toc;
disp(['Captura finalizada en ' num2str(tiempoTranscurrido, '%.2f') ' segundos.']);
disp('Generando gráfica...');

% --- 6. Graficación de Datos ---
figure;
plot(tiempo, dataMatrix(1:length(tiempo),1), 'b', 'LineWidth', 1.5); hold on;
plot(tiempo, dataMatrix(1:length(tiempo),2), 'r', 'LineWidth', 1.5);
xlabel('Tiempo (s)');
ylabel('Voltaje (V)');
title('Comportamiento entrada-salida del sistema');
legend('Entrada u(t)', 'Salida y(t)', 'Location', 'best');
grid on;

disp('Proceso finalizado.');
clear s;
