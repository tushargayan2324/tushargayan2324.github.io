# tushargayan2324.github.io
Personal Website


function []=datacheck(flight_condition)
% DATACHECK(FLIGHT_CONDITION) where FLIGHT_CONDITION is one of [101,
% 106, 109]. Compute coherence between streamwise sensor responses and
% a reference sensor. Produces figure similar to figure 9(b) from
% Palumbo paper.

  if nargin<1, flight_condition = 101; end

  % constants - adjust nfft, nkeep as needed
  nfft   = 8192; 
  fs     = 204800;
  nkeep  = 2^20; 
  refMic = 1;
  fplot  = 1500;
  hwin   = hann(nfft);
  
  % channels and sensor spacing. 
  channels = [44:-1:36,1:27,29:33,34,35];
  iref     = find(channels==refMic);
  xlocs    = cumsum([ 0 ones(1,4)*0.012 ones(1,5)*0.006 ones(1,19)*0.003 ...
  		ones(1,5)*0.006 ones(1,2)*0.012 ones(1,5)*0.036 0.072 0.036]);
  
  % load data, compute coherence with ref mic
  testDir = ['.',filesep,'B',num2str(flight_condition),filesep];
  refData = load([testDir 'Channel' num2str(channels(iref)) '.mat']);
  refData = refData.singleData(1,1:nkeep).';
  
  fprintf('Channel:\n');
  coh = ones(length(channels),1);
  for ind = 1:length(channels),
    if ind==iref, continue, end
    tmp = load([testDir 'Channel' num2str(channels(ind)) '.mat']);
    if mod(ind,8), fprintf('%02d ',channels(ind)); else fprintf('%02d\n',channels(ind)); end
    [ctmp,~] = mscohere(refData,tmp.singleData(1,1:nkeep).',hwin,nfft/2,[fplot fplot+10],fs);
    coh(ind,1) = ctmp(1);
  end
  fprintf('\n');
  
  % plot coherence
  figure; plot(xlocs - xlocs(iref),coh,'-o','linewidth',2);
  set(gca,'fontsize',14);
  xlabel(sprintf('Distance from mic %d (m)',refMic)); ylabel('\gamma^2');
  title(sprintf('Coherence at %d Hz',fplot));
  
