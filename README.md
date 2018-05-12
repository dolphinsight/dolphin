Dolphinsight Developer Guide, V1.0

by kingaim

1. About Dolphinsight

Dolphinsight(DI) is an interactive big data viz application which supports diverse data sources.
The target users for DI includes: data scientist, BI developer and anyone work with data and eager to find the 
data insight via simply interactive drag-n-drop manner.

Dolphinsight is developed by Dolphinsight Team(DIT). Please contact DIT via dolphinsight.bigdata@gmail.com.
More information about Dolphinsight and DIT, please visit www.github.com/dolphinsight

2. Prerequisites

3. Dolphinsight Architecture

Currently, the DI is made up three key components and they dolphinsight(core), dolphinprotocol(protocol) and 
dolphinteractive(interactive).

The core holds all key modules for the DI as the backend.
The interactive is the UI frontend.
The protocol defines the "language" speek between core and interactive, and it works quite simply as Google Protocolbuffer.



