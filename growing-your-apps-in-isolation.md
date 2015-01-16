# Growing Your Apps in Isolation

---

author : Peter Burrows

---

![](http://resource.docker.cn/tech_docker52__01__970-630x420.jpg)


Internet software has made life easier for most of us, but not for the IT staffers who have to keep it up and running. Whether an app is designed to summon a cab, check the weather, or search job postings, it has to be painstakingly tested and configured so it works even during upgrades and other potentially disruptive changes. Many companies spend more than 90 percent of their IT budgets on such thankless scut work, says Al Gillen, an analyst at researcher IDC.

That helps explain the rise of Docker. In less than two years the company’s free software has become a building block in app development at thousands of companies, and it’s supported by Amazon.com (AMZN), Google (GOOG), IBM (IBM), Microsoft (MSFT), and VMware (VMW). Docker automatically rearranges programmers’ code into virtual “containers” that are preconfigured and standardized to work on most any hardware setup, shaving weeks or months off the process of sending an app into the world. “This is about the mass commoditization of the production of software,” says founder Solomon Hykes, who named the company and its software after longshoremen. “Docker can have the same impact on software that shipping containers had on world trade.”

IBM, Google, and other companies had designed and used their own container technologies for years, but Hykes was the first to establish an industry standard. He did that by creating an easy-to-use interface and by making Docker an open-source project, freeing other tech companies that contributed code to create software and services they could sell to Docker users. Although Hykes and Chief Executive Officer Ben Golub have yet to demonstrate that they can make money selling versions of the software with extra features, they’ve raised $55 million in venture capital, including a $40 million round in September that valued the company at more than $400 million, says a person familiar with the financing who wasn’t authorized to discuss it publicly. “There’s an awful lot of momentum behind Docker, because it solves a lot of problems the industry has had for a long time,” says IDC’s Gillen.

Hykes, a 30-year-old Frenchman with a master’s degree in computer science from the European Institute of Technology, got the idea for Docker while working in a string of jobs, where he had to spend most of his time writing app-launching software over and over. In 2007 he quit a \$40,000-a-year job in Los Angeles and moved back to his mom’s basement in Paris to work on his pet project with friends. He and one of his pals landed a spot in the 2010 class of Y Combinator, the San Francisco startup accelerator. They launched a company called DotCloud and, after raising about $10 million in venture capital, decided to stay in the U.S. “He’s always known where he wanted to go,” says Docker board member Marc Vestraen, an Apple ([AAPL](http://investing.businessweek.com/research/stocks/snapshot/snapshot.asp?ticker=AAPL)) engineering manager. The two met through Tay Son Vo Dao, a Vietnamese martial arts club, and still try to practice together twice a week.

![](http://resource.docker.cn/tech_docker52__03__630x420.jpg)

DotCloud earned technical accolades but struggled to find customers. In April 2013, Hykes announced at PyCon, an industry conference, that he’d converted the software, now called Docker, into an open-source project. Within hours, dozens of companies began to experiment with Docker and contribute code to improve it. Programmers at hundreds of companies, from little-known startups to Goldman Sachs ([GS](http://investing.businessweek.com/research/stocks/snapshot/snapshot.asp?ticker=GS)), have downloaded the free code more than 77 million times, the company says. Hykes and his wife, Coralie, a fellow martial artist and a biotech engineer who became a florist when the couple moved to San Francisco, have swapped their tiny apartment for a row house in the trendy NoPa (North of Panhandle Park) neighborhood. Hykes recently replaced his broken-down 2001 Volkswagen Golf with the 2014 model.

Millions of downloads are one thing, commercialization another. Docker, which now has 70 employees in an office stuffed with picnic tables, ferns, and a pet tortoise, has less than $10 million in annual revenue, says the person familiar with its finances. The company’s free roots make the job tougher, says Luke Kanies, who created a popular open-source system called Puppet. For companies like his, Kanies says, “the major thing people love about you is that you’re free.”

![](http://resource.docker.cn/tech_docker52__02__630x420.jpg)

For Hykes, that problem became clear when Docker took its first sizable step toward building a business on Dec. 4, introducing a subscription-based online service called Docker Hub Enterprise aimed at managing containers for large companies. On the same day, CoreOS CEO Alex Polvi, who built his business selling server software designed to work well with Docker, blogged that Docker’s business model was “fundamentally flawed” and that the industry should coalesce behind his company’s new container software, called Rocket. Furious, Hykes issued a series of emotional tweets in response, calling Rocket “a copy-paste of our own explicit design roadmap” and accusing CoreOS of trying to manufacture controversy. Hykes has since apologized. “I’d give myself an ‘F’ on that. I overreacted,” he says.

The Docker founder says he’s not worried about rival container software but acknowledges that he has work to do to persuade other companies to continue contributing code to Docker and developing products for its users as the company starts trying to turn a profit. “The onus is on us,” he says, “to prove we’re not out to steal anyone’s business but to create a level playing field.”
