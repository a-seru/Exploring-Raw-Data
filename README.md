# Exploring-Raw-Data
Exploring the 5 classes of Fault based on its Fault intensity
Outdoor air temperature sensor bias (Bias= 2 deg, Bias= 4 deg, Bias= -2 deg, Bias= -4 deg)
Join = vertcat(oabias2annual,oabias2annual1,oabias4annual,oabias4annual1)

% All desired columns to remove
colsToRemove = {'OA_CFM', 'RA_CFM', 'SA_CFM', ...
                'SF_SPD', 'RF_SPD', ...
                'SF_WAT', 'RF_WAT', ...
                'OA_DMPR', 'RA_DMPR', ...
                'CHWC_VLV', 'SYS_CTL'}

% Find intersection between existing column names and the list to remove
existingCols = intersect(colsToRemove, Join.Properties.VariableNames)

% Remove only those columns that actually exist
Join(:, existingCols) = []

% Assume your table is named Join
% If there's no datetime column, create one:
n = height(Join)
Join.Datetime = datetime(2025,1,1) + minutes(0:n-1)
% 
% Convert FAULT_TYPE to numeric values
[uniqueFaults, ~, faultCodes] = unique(Join.FAULT_TYPE)
Join.FAULT_TYPE = faultCodes
% 
% % Variables to loop through
variables = {'ZONE_TEMP_1', 'ZONE_TEMP_2', 'ZONE_TEMP_3', ...
             'ZONE_TEMP_4', 'ZONE_TEMP_5', 'SF_CS', 'SF_SPD_DM','CHWC_VLV_DM','MA_TEMP','OA_TEMP','RA_DMPR_DM','RA_TEMP','RF_SS','RF_SPD_DM','SA_SP','SA_SPSPT','SA_TEMPSPT','SA_TEMP'};
% 
% % Loop through each variable
for i = 1:length(variables)
    figure
    var = variables{i}
% 
%     % Extract data for current plot
    x = Join.Datetime
    y = Join.(var)
    z = Join.FAULT_TYPE
% 
%     % Plot 3D scatter
    scatter3(x, y, z, 50, z, 'filled') % Color by fault code
    xlabel('Datetime')
    ylabel(var)
    zlabel('Fault Type Code')
    title(['3D Plot of ', var, ' vs Datetime and Fault Type'])
    grid on
    colormap jet
    colorbar('Ticks', 1:length(uniqueFaults), 'TickLabels', uniqueFaults)
% 
%     % Improve datetime formatting on X axis
    datetick('x','yyyy-mm-dd HH:MM','keepticks')
    rotate3d on
end
--------------------------------------------------------------------------------------------------------------------------
Outdoor Air Damper (10%, 25%, 75%, 100%)
Join = vertcat(damperstuck010annual,damperstuck025annual,damperstuck075annual1)

colsToRemove = {'OA_CFM', 'RA_CFM', 'SA_CFM', ...
                'SF_SPD', 'RF_SPD', 'SF_WAT', 'RF_WAT', ...
                'OA_DMPR', 'RA_DMPR', 'CHWC_VLV', 'SYS_CTL'}

% Remove only if the columns exist (prevents error)
colsToRemove = intersect(colsToRemove, Join.Properties.VariableNames)

Join = removevars(Join, colsToRemove)

% Downsample (optional)
Join_ds = Join(1:100:end, :)  % Adjust step as needed

% List of variables to visualize (replace with actual column names)
sensorVars = {'SA_TEMP', 'MA_TEMP', 'ZONE_TEMP_1', 'ZONE_TEMP_2', ...
              'ZONE_TEMP_3', 'ZONE_TEMP_4', 'ZONE_TEMP_5','CHWC_VLV_DM','RA_DMPR_DM','RA_TEMP','RF_SS','RF_SPD_DM','SA_SP','SA_SPSPT','SA_TEMPSPT','SF_CS','SF_SPD_DM','OA_TEMP'}

% Convert FAULT_TYPE to categorical for grouping
faultTypes = categorical(Join_ds.FAULT_TYPE)
uniqueFaults = categories(faultTypes)
colors = lines(numel(uniqueFaults))  % Distinct colors for each fault type

for i = 1:length(sensorVars)
    figure
    hold on

    for j = 1:numel(uniqueFaults)
        idx = faultTypes == uniqueFaults{j}
        scatter3(Join_ds.Datetime(idx), ...
                 Join_ds.(sensorVars{i})(idx), ...
                 repmat(str2double(regexp(uniqueFaults{j}, '\d+', 'match', 'once')), sum(idx), 1), ...
                 15, colors(j,:), 'filled', 'DisplayName', uniqueFaults{j})
    end

    xlabel('Datetime')
    ylabel(sensorVars{i})
    zlabel('Fault Severity (%)')
    title(['3D Plot: ', sensorVars{i}, ' vs Time'])
    legend('Location','bestoutside')
    view(135, 30)
    grid on
end
--------------------------------------------------------------------------------------------------------------------------
Cooling Coil Valve (10%, 25%, 40%, 50%)
% --- Step 1: Prepare ---
% Get variable names to plot (exclude Datetime and FaultType)
allVars = Combine_N.Properties.VariableNames;
featureVars = setdiff(allVars, {'Datetime', 'FaultType'});  % Features to plot

% Convert FaultType to numeric codes
[faultLabels, ~, faultIDs] = unique(Combine_N.FAULT_TYPE);  % faultIDs: 1, 2, 3, ...

% Downsample to reduce memory use
downsampleRate = 60;
sampleIdx = 1:downsampleRate:height(Combine);

% Convert datetime to numeric for plotting
timeNumeric = datenum(Combine_N.Datetime(sampleIdx));
faultNumeric = faultIDs(sampleIdx);

% --- Step 2: Loop through all features ---
for i = 1:length(featureVars);
    featureName = featureVars{i};
    featureData = Combine.(featureName)(sampleIdx);

    % --- 3D Plot ---
    figure('Name', featureName, 'NumberTitle', 'off');
    scatter3(timeNumeric, faultNumeric, featureData, 10, faultNumeric, 'filled');  % Color by fault ID

    % Labels and styling
    datetick('x', 'keepticks');
    xlabel('Datetime');
    ylabel('Fault Type (Numeric)');
    zlabel(featureName, 'Interpreter', 'none');
    title(['3D Plot of ', featureName, ' over Time and Fault Type'], 'Interpreter', 'none');
    grid on;
    view(45, 25);

    % --- Colorbar with fault labels ---
    colormap(lines(length(faultLabels)));  % Use distinct colors
    cb = colorbar;
    caxis([1, length(faultLabels)]);       % Ensure colorbar covers all faults
    cb.Ticks = 1:length(faultLabels);
    cb.TickLabels = cellstr(faultLabels);  % Display actual fault names
    cb.Label.String = 'Fault Type';
end;
--------------------------------------------------------------------------------------------------------------------------
Comparing the Faulty data to the Fault-free dataset (Faulty data that contains sensor readings)
Outdoor air temperature sensor bias & Supply air temperature sensor bias vs Fault Free
Combine = vertcat(HEALTHY,oabias4annual1,oabias4annual,oabias2annual1,oabias2annual,sabias4annual1,sabias4annual,sabias2annual1,sabias2annual)

columnsToRemove = {'OA_CFM', 'RA_CFM', 'SA_CFM', 'SF_SPD', ...
                   'SF_WAT', 'RF_WAT', 'OA_DMPR', 'RA_DMPR', ...
                   'CHWC_VLV', 'SYS_CTL'}

Combine = removevars(Combine, columnsToRemove)

% Downsample (if needed)
Combine_ds = Combine(1:100:end, :)  % Adjust step as needed

% Convert datetime if necessary
if ~isdatetime(Combine_ds.Datetime)
    Combine_ds.Datetime = datetime(Combine_ds.Datetime)
end
timeNum = datenum(Combine_ds.Datetime)

% List of selected variables to plot
selectedVars = {'ZONE_TEMP_1', 'ZONE_TEMP_2', 'SA_SPSPT', 'SA_SP', ...
                'OA_TEMP', 'RA_TEMP', 'MA_TEMP', 'CHWC_VLV_DM', ...
                'RF_SPD_DM', 'RF_SS',"CHWC_VLV_DM",'MA_TEMP',"OA_TEMP",'RA_DMPR_DM','RF_SPD_DM','SA_SPSPT','SA_TEMP','SA_TEMPSPT','SF_CS','SF_SPD_DM','ZONE_TEMP_3','ZONE_TEMP_4','ZONE_TEMP_5'}  % Add more as needed

% Extract fault types
faultTypes = Combine_ds.FAULT_TYPE
if iscell(faultTypes)
    faultTypes = categorical(faultTypes)
elseif isstring(faultTypes)
    faultTypes = categorical(faultTypes)
end
uniqueFaults = categories(faultTypes)

% Create color map: assign black to 'Fault Free', random distinct colors to others
colors = lines(numel(uniqueFaults))
for i = 1:length(uniqueFaults)
    if strcmp(uniqueFaults{i}, 'Fault Free')
        colors(i, :) = [0 0 0]  % black
    end
end

% Loop through each selected variable
for v = 1:length(selectedVars)
    varName = selectedVars{v}

    if ~ismember(varName, Combine_ds.Properties.VariableNames)
        warning('%s not found in table.', varName)
        continue
    end

    yData = Combine_ds.(varName)

    figure
    hold on

    % Plot each fault type with its color
    for i = 1:length(uniqueFaults)
        idx = faultTypes == uniqueFaults{i};
        scatter3(timeNum(idx), repmat(v, sum(idx), 1), yData(idx), ...
                 36, colors(i,:), 'filled', ...
                 'DisplayName', char(uniqueFaults{i}))
    end

    xlabel('Datetime')
    ylabel('Variable Index')
    zlabel('Value')
    title(sprintf('3D Visualization for %s by Fault Type', varName))
    datetick('x', 'keeplimits')
    legend show
    view(45, 30)
    grid on
end
--------------------------------------------------------------------------------------------------------------------------
Comparing the Faulty data to the Fault-free dataset 
OA Damper & Cooling Coil Valve vs Fault Free
Combine = vertcat(HEALTHY,coistuck075annual,coistuck050annualfinal,coistuck010annual,coistuck025annual,coileakage050annual,coileakage040annual,coileakage025annual,coileakage010annual,damperstuck075annual1,damperstuck025annual,damperstuck010annual)

columnsToRemove = {'OA_CFM', 'RA_CFM', 'SA_CFM', ...
                   'SF_SPD', 'SF_WAT', 'RF_WAT', ...
                   'OA_DMPR', 'RA_DMPR', 'CHWC_VLV', 'SYS_CTL'}

Combine = removevars(Combine, columnsToRemove)

% Downsample
dsFactor = 100
Combine_ds = Combine(1:dsFactor:end, :)

% Convert FAULT_TYPE to double if needed
if iscategorical(Combine_ds.FAULT_TYPE)
    Combine_ds.FAULT_TYPE = double(Combine_ds.FAULT_TYPE)
end

% Plot variables
plotVars = {'ZONE_TEMP_1', 'ZONE_TEMP_2', 'ZONE_TEMP_3', ...
            'SA_SP', 'ZONE_TEMP_4', 'ZONE_TEMP_5', ...
             'SA_TEMP','SA_TEMPSPT', 'SA_SPSPT', ...
             'RF_SS', 'OA_TEMP', 'RA_TEMP', ...
            'RA_DMPR_DM', 'CHWC_VLV_DM', 'MA_TEMP', ...
            'SF_SPD_DM','SF_CS', 'RF_SPD_DM'}

% Colors setup
faultTypes = unique(Combine_ds.FAULT_TYPE);
nFaults = length(faultTypes)
colors = lines(nFaults)  % Predefined colormap

% Define fault labels
faultTypeLabels = containers.Map(...
    {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11}, ...
    {'Fault Free', ...
     'coileakage010annual', ...
     'coileakage025annual', ...
     'coileakage040annual', ...
     'coileakage050annual', ...
     'coistuck010annual', ...
     'coistuck025annual', ...
     'coistuck050annual', ...
     'coistuck075annual', ...
     'damperstuck010annual', ...
     'damperstuck025annual', ...
     'damperstuck075annual'})

% Plotting loop
for i = 1:length(plotVars)
    figure
    hold on

    for j = 1:length(faultTypes)
        ft = faultTypes(j);
        dataSubset = Combine_ds(Combine_ds.FAULT_TYPE == ft, :)

        % Assign black color for Fault Free
        if ft == 0
            color = [0, 0, 0]  % Black
        else
            color = colors(j, :)  % From colormap
        end

        % Get label from map
        if isKey(faultTypeLabels, ft)
            label = faultTypeLabels(ft)
        else
            label = ['Fault Type ' num2str(ft)]
        end

        scatter3(dataSubset.Datetime, ...
                 dataSubset.(plotVars{i}), ...
                 dataSubset.FAULT_TYPE, ...
                 20, color, 'filled', ...
                 'DisplayName', label)
    end

    title(plotVars{i}, 'Interpreter', 'none')
    xlabel('Datetime')
    ylabel(plotVars{i})
    zlabel('FAULT TYPE')
    legend('show', 'Location', 'bestoutside')
    grid on
    view(45, 30)
    hold off
end





