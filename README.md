# nosql-data-cache
Faster data look up. Can be easily used in any Spring based projects.

##Why should I use this? Why can't I leverage from existing cache technologies?
  
  This API addresses a special use case. Normally the cache APIs works by caching the results of a particular query after loading the results for the first time from database. We don't want to make this queries once the business processing is started. In our case, we load the entire data initially and make the queries to this in memory cache instead to database. 

Do you access data from  database frequently to acheive a business process?

Did you try fine tuning your databae/jpql many times and still code is not faster?

Do you have a large file that you have to iterate and identify some records frequently?

Try this nosql-data-cache and this will simplify the look up.

This API will help you to load and index the minimal data required to acheive a business process based on the look up queries.  

Once you done with the cache,release the cache. 

With 200K records in cache to look up all records with five look up queries, it took about 150 milli seconds.

Java version: JDK 7 or above. 

           <dependency>
			<groupId>com.github.ledwinson</groupId>
			<artifactId>nosql-data-cache</artifactId>
			<version>1.0.0</version>
		</dependency>

### Test it before using it and see how it fits to your use case.
  This cache mainly depedens on two things, 
  1) one is the data and 
  2) second is the look up queries.   
  
  The data can be loaded to cache by passing an implementation of DataSourceResolver<T>.  
  The lookup queries can be declared in an interface exting from Snapshot interface. The method names must start with findBy

### As an example here I am loading data from a CSV file convert it to member obejcts and trying to look up members based on the queries defined in test snapshot.

### Step1 Inject the cache dependency.

    `@Autowired`
    `private CacheService<Membership, TestSnapshot, Long> cacheService;`
    
    Here Membership is the entity [or DTO] for my data, TestSnapshot is the interface with look up queries and Long is the data type of primary key. Note that here I loaded data from CSV file and there is no primary key associated with it, so a Long field is created [memberId] and just incremented the numbers.
    
###  Load the complete data required to do the look up.

        `final PrimaryKeyType<Long> keyType = new PrimaryKeyType<Long>(Long.class, "memberId", Membership.class);`
        `final int hundred = 100;`
        `cacheService.cache(new SampleDataSource(hundred ), false, keyType , TestSnapshot.class);`
        
        Note that if you have very large set of data, you can do this when the user logged in to your system in a different thread.
        SampleDataSource is the implementation of DataSourceResolver<T> interface that can read a file [or database] and create a collection of membership objects. 
    
### Once the data is loaded the snapshot can be obtained. You can call this method any time, this is a blocking call.
`final TestSnapshot snapshot = cacheService.getSnapshot();`

### That's all, now look up your specific data,
`final Membership memberFromCache = snapshot.findByMemberNumberAndTfn(1234, 10);`

 //release snapshot after done with it.
 `snapshot.release();`

### TestSnapshot interface

`import java.util.Date;`
`import java.util.List;`

`import le.cache.bis.services.Snapshot;`
`import le.cache.bis.services.impl.data.Membership;`
`import le.cache.util.Property;`
`public interface TestSnapshot extends Snapshot {`

    // TI am sure there is pnly one record for this query and employee is an entity liked with membership entity.
    `Membership findByMemberNumberAndTfnAndDob(@Property("memberNumber") String memberNumber, `
            `@Property("superFundGenEmprId") String tfn, @Property(value = "dob", on = "employee") Date dob);`

    
    `Membership findByMemberNumberAndTfnForEmployer(@Property("memberNumber") String memberNumber,  @Property("superFundGenEmprId") String tfn);`
    
    //expecting multiple results.
    `List<Membership> findByMemberNumberAndTfnForEmployerForDuplicates(@Property("memberNumber") String memberNumber,  @Property("superFundGenEmprId") String tfn);`
`}`


## SampleDataResolver Class

`public class SampleDataSource implements DataSourceResolver<Membership>{`
    `private final int size;`
    
    `public SampleDataSource(int size) {`
        `this.size = size;`
    `}`
        
    `@Override`
    `public Collection<Membership> load() {`
        `return getTestData();`
    `}`
    
    `private Collection<Membership> getTestData() {`
        `final List<Membership> members = csvUtil.getmembers();`
        return members;
    `}`
    `}`
