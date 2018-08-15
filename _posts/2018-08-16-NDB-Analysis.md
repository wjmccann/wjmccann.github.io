---
layout: post
title: Notifiable Data Breach Analysis
tags: [Security]
---
On the 22nd of February 2018, the Australian Government introduced the Notifiable Data Breaches scheme.  The scheme falls under the Privacy Act 1988 and establishes the requirements for entities when responding to data breaches. Essentially, the requirement is for entities (business, government organisations, etc) to provide notification to the Australian Government when a data breach has occurred that results in serious harm to any individual whose personal information was involved in that data breach. 

The [Notifiable Data Breach Scheme website]( https://www.oaic.gov.au/privacy-law/privacy-act/notifiable-data-breaches-scheme) contains a lot of good information on the requirements of the scheme. In addition to the scheme, the Office of the Australian Information Commissioner (OAIC) produce quarterly statistics on breaches that are reported. The blog post analyses the most recent report which covers the period 1st of April to 30th June 2018. 
## Key Statistics

During the period of the report, there were a total of 242 notifications. In the first quarterly report there were a total of 63 notifications. Since the scheme was introduced in February there has been a steady increase in notifications, starting with 8 in February through to 90 in June. 

![]( https://www.oaic.gov.au/resources/privacy-law/privacy-act/notifiable-data-breaches-scheme/quarterly-statistics/2018-04/chart1.1.png)

The break down of the latest statistics is interesting and show that 59% of the data breaches reported were due to malicious or criminal attacks, 36% were human error and 5% were system faults.

![]( https://www.oaic.gov.au/resources/privacy-law/privacy-act/notifiable-data-breaches-scheme/quarterly-statistics/2018-04/chart1.4.png)

## Scale of the breaches
The following chart does a good job of detailing the size of the breaches that occurred. 

![]( https://www.oaic.gov.au/resources/privacy-law/privacy-act/notifiable-data-breaches-scheme/quarterly-statistics/2018-04/chart1.2.png)

The interesting thing is that the largest number size of breach, was in the 11-100 scale. This is relatively small. It should also be noted that the bulk of the breaches for between 1000 through to 1 individual affect. So, what does this mean? Looking at the data it suggests that organisations are getting better at reporting breaches, regardless of the size of the incident they are still reporting. Sadly, the data doesn’t show a closer break down of the statistics, so there is no way to determine if the smaller breaches are simply human error and the larger ones are all criminal activity. 

Additionally, the type of information that is breached is interesting. The bulk of the information breached is Contact Information at 89% of information involved in breaches. Noting that 56% of the breaches are malicious or criminal activity, this suggests that for the most part malicious actors have not gained significant information (not to down play contact information, but it is far low on the scale when compared to Tax File Numbers of credit card information)

![]( https://www.oaic.gov.au/resources/privacy-law/privacy-act/notifiable-data-breaches-scheme/quarterly-statistics/2018-04/chart1.3.png)

## Malicious Actor Breakdown
The following image demonstrates the breakdown of all malicious or criminal activity across all sectors that submitted data. The most significant event is a cyber incident. The term cyber incident is used to describe an incident in which a malicious actor has breached the digital security of an organisation. 

![]( https://www.oaic.gov.au/resources/privacy-law/privacy-act/notifiable-data-breaches-scheme/quarterly-statistics/2018-04/chart1.6.png)

The breakdown of the Cyber Incident is interesting reading and provides some good insight into the types of threats Australian entities are facing today. 

![]( https://www.oaic.gov.au/resources/privacy-law/privacy-act/notifiable-data-breaches-scheme/quarterly-statistics/2018-04/chart1.7.png)

It is very evident that the majority of access which lead to a data breach was through the use and abuse of credentials. Over 77% of Cyber Incidents involve credentials. 34% of the incidents are compromised or stolen credentials through an unknown method. We are left to only speculate, however I wouldn’t be surprised if this was through using existing breach data and users reusing passwords. In a penetration testing engagement this is one of the first tasks to gather all known passwords that may exist in a previous breach. It is unclear what is defined by a brute-force attack with compromised credentials – Are the existing data breach credentials or stolen credentials? Again, we can only speculate. Sadly, the statistics also demonstrate that Phishing campaigns are still effective when gaining credentials and access and this is also confirmed through penetration testing engagements where I have seen success (in terms of gaining credentials) in all phishing attempts made to date. 

Again, we can only speculate on what the 10% of Hacking (other means) is. My assumption this the abuse of misconfigurations, or out of date software with exploits. The low number of Ransomware and Malware makes sense, in that these types of attacks are not typically designed to gain data from an organisation and are designed to elicit money from the organisation. 

## Breakdown by Sector. 
Of the 242 breach notifications, the top two sectors which generated notifications were Health service providers (49 breaches) and the Finance sector (36 breaches). 

When these sectors are broken down it shows some interesting data. For example, the Health Sector has 29 breaches which were caused by Human Error. This equates to almost 60% of all their breaches reported. Human Error data breaches can be as simple as sending an email to the incorrect people or not using BCC. When you look at the information that Health service providers are dealing with, you can easily see why they would have so many Human Error breaches. 

The finance sector has a similar mix with 50% Human error, 47% malicious or criminal attack and 3% system fault. When breaking down the finance sectors malicious or criminal attacks they have 14 cyber incident breaches compared to the 8 that the health sector has. This indicates that the finance sector is more heavily targeted by malicious actors when compared to the Health sector. Interestingly of the 14 cyber incidents in the finance sector, 7 of these were due to phishing and 5 were through compromised or stolen credentials (method unknown). Again, this points to the fact that the finance sector is targeted more. The reason for this is most likely due to the fact that financial information is a very juicy target. This also provides some insight into who the attacker may be – it is most likely not a nation-state actor and more likely to be a criminal organisation targeting the financial sector.

![]( https://www.oaic.gov.au/resources/privacy-law/privacy-act/notifiable-data-breaches-scheme/quarterly-statistics/2018-04/chart2.4.png)

## Conclusion
The Quarterly statistics for the Notifiable Data Breach Scheme are an interesting read and provide good insight into the types of threats that Australian Entities are facing today. The dramatic increase in notifiable breaches in this quarter compared to the last isn’t a good indication of whether cyber attacks on Australia are increasing or decreasing. Instead it demonstrates a level of maturity in entities as they become more comfortable in reporting incidents, no matter how small. It will be interesting to see in the next quarter if the statistics increase or stay the same. 

