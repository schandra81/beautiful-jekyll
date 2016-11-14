---
layout: post
published: true
title: Running Multiple threads in queue using BlockingCollections
subtitle: Continuously running different tasks one by one with a Queue mechanism.
date: '2016-10-12'
---
My program has 3 functions. Each function takes a list of Items and fill certain information. For example

```
class Item {
 String sku,upc,competitorName;
 double price;
}
```
function F1 takes a List and fills upc

function F2 takes List (output of F1) and fills price.

function F3 takes List (output of F2) and fills competitorName

F1 can process 5 items at a time, F2 can process 20 items at a time, F3 also 20.

Right now I am running F1 -> F2 -> F3 in serial because F2 needs info(UPC code) from F1. F3 needs price from F2.

I would like to make this process efficient by running F1 run continuously instead of waiting for F2 and F3 to be completed. F1 executes and output into queue then F2 takes 20 items at a time and process them. and then follows F3.

How can i achieve this by using BlockingCollection and Queue?

```csharp
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

namespace BlockingCollectionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            BlockingCollection<Listing> needUPCJobs = new BlockingCollection<Listing>();
            BlockingCollection<Listing> needPricingJobs = new BlockingCollection<Listing>();

            // This will have final output
            List<Listing> output = new List<Listing>();

            // start executor 1 which waits for data until available
            var executor1 = Task.Factory.StartNew(() =>
            {
                int maxSimutenousLimit = 5;
                int gg = 0;
                while (true)
                {
                    while (needUPCJobs.Count >= maxSimutenousLimit)
                    {
                        List<Listing> tempListings = new List<Listing>();
                        for (int i = 0; i < maxSimutenousLimit; i++)
                        {
                            Listing listing = new Listing();
                            if (needUPCJobs.TryTake(out listing))
                                tempListings.Add(listing);
                        }
                        // Simulating some delay for first executor
                        Thread.Sleep(1000);

                        foreach (var eachId in tempListings)
                        {
                            eachId.UPC = gg.ToString();
                            gg++;
                            needPricingJobs.Add(eachId);
                        }
                    }

                    if (needUPCJobs.IsAddingCompleted)
                    {
                        if (needUPCJobs.Count == 0)
                            break;
                        else
                            maxSimutenousLimit = needUPCJobs.Count;
                    }                    
                }
                needPricingJobs.CompleteAdding();
            });

            // start executor 2 which waits for data until available
            var executor2 = Task.Factory.StartNew(() =>
            {
                int maxSimutenousLimit = 10;
                int gg = 10;
                while (true)
                {
                    while (needPricingJobs.Count >= maxSimutenousLimit)
                    {
                        List<Listing> tempListings = new List<Listing>();
                        for (int i = 0; i < maxSimutenousLimit; i++)
                        {
                            Listing listing = new Listing();
                            if (needPricingJobs.TryTake(out listing))
                                tempListings.Add(listing);
                        }
                        // Simulating more delay for second executor
                        Thread.Sleep(10000);

                        foreach (var eachId in tempListings)
                        {
                            eachId.Price = gg;
                            gg++;
                            output.Add(eachId);
                        }
                    }
                    if (needPricingJobs.IsAddingCompleted)
                    {
                        if(needPricingJobs.Count==0)
                            break;
                        else
                            maxSimutenousLimit = needPricingJobs.Count;
                    }
                }

            });

            // producer thread
            var producer = Task.Factory.StartNew(() =>
            {
                for (int i = 0; i < 100; i++)
                {
                    needUPCJobs.Add(new Listing() { ID = i });
                }
                needUPCJobs.CompleteAdding();
            });

            // wait for producer to finish producing
            producer.Wait();

            // wait for all executors to finish executing
            Task.WaitAll(executor1, executor2);

            Console.WriteLine();
            Console.WriteLine();
        }
    }

    public class Listing
    {
        public int ID;
        public string UPC;
        public double Price;
        public Listing() { }
    }
}
```
