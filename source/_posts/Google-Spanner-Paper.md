---
title: Google-Spanner-Paper
date: 2019-06-10 07:57:01
tags:
---

**Spanner: Google's Globally Distributed Database**

JAMES C.CORBETT, JEFFREY DEAN, MICHAEL EPSTEIN, ANDREW FIKES, CHRISTOPHER FROST,  J. J. FURMAN, SANJAY GHEMAWAT, ANDREY GUBAREV, CHRISTOPHER HEISER, PETER HOCHSCHILD, WILSON HSIEH, SEBASTIAN KANTHAK, EUGENE KOGAN, HONGYI LI, ALEXANDER LLOYD, SERGEY MELNIK, DAVID MWAURA, DAVID NAGLE, SEAN QUINLAN, RAJESH RAO, LINDSAY ROLIG, YASUSHI SAITO, MICHAL SZYMANIAK, CHRISTOPHER TAYLOR, RUTH WANG, and DALE WOODFORD, Google, Inc.

Spanner is Google's scalable, multiversion(多版本), globally distributed, and synchronously replicated database. It is the first system to distribute data at global scale and support externally-consistent distributed transactions. This article describes how Spanner is structured, its feature set, the rationale(理论基础) underlying various design to supporting external consistency and a variety of powerful features: nonblocking reads in the past, lock-free snapshot, and atomic shcema changes, across all of Spanner.

Categories and Subject Descriptors. H.2.4 [Database Management]：Systems-Concurrency, distributed databases, transaction processing

General Terms: Design, Algorithms, Performance

Additional Ket Words and Phrases: Distributed databases, concurrency control, replication, transactoins, time management

**ACM Reference Format:** 

Corbett, J. C., Dean, J., Epstein, M., Fikes, A., Frost, C., Furman, J. J., Ghemawat, S., Gubarev, A., Heiser, C., Hochschild, P., Hsieh, W., Kanthak, S., Kogan, E., Li, H., Lloyd, A., Melnik, S., Mwaura, D., Nagle, D., Quinlan, S., Rao, R., Rolig, L., Saito, Y., Szymaniak, M., Taylor, C., Wang, R., and Woodford, D. 2013. Spanner: Google’s globally distributed database. ACM Trans. Comput. Syst. 31, 3, Article 8 (August 2013), 22 pages.

DOI:http://dx.doi.org/10.1145/2491245

