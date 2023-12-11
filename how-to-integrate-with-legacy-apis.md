
# How to Successfully Integrate with Legacy APIs Using NodeJS

Hello, I'm Aluisio, a software engineer with a rich history of integrating with Brazilian banks, insurance companies, and ERPs.

In this article, I'll share my experience and guide you through the process of seamless integration. For instructional purposes, I'll exaggerate partner inefficiencies, but it's crucial to tailor these strategies to your unique context.

## Document Everything
When embarking on the integration journey, the initial step is to request documentation. However, the reality is that **not every company excels at documentation**.

- Uncertainty looms over the structure of responses.
- The configuration of the POST body remains a mystery.
- Required parameters are shrouded in ambiguity.
- The meanings of various fields are unclear.

**The solution? Write your documentation!**

Begin by selecting your preferred platform for API development (e.g., Postman, Insomnia), then create a collection and commence drafting your requests.

> If feasible, consider importing the requests provided by your partner.
![Postman printscreen](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fu7je4hh1ngh5uh7d6tu.png)

**Create a Dedicated Section for Your Documentation**

Establish a dedicated section for your documentation, whether it's a [Wiki](https://docs.github.com/pt/communities/documenting-your-project-with-wikis/about-wikis) within your repository, a 'docs' folder in your monorepo, a page on Notion, or any other suitable platform.

This documentation should encompass the following key components:

**Docs**: Include links to the partner's documentation. This serves as a quick reference for your team to access essential information.

**Contacts**: Documenting available contacts is crucial. Specify people's names and their respective positions. This not only provides clarity on the support network but also ensures that in the absence of the designated integration contact, another developer can seamlessly take over.

**Working Requests**: Compile a comprehensive list of all endpoints that are functional and yield valuable data for your business. Each entry in this list should include a detailed description (as interpreted by your team) of the endpoint, its required parameters, and a breakdown of each response field. For any unknown fields, mark them with a 'TODO,' and ensure that the relevant partner contacts are activated for swift resolution.

This structured documentation will significantly enhance your integration process, fostering clarity and efficiency within your team.

## Write an RFC (Request for Comments)

Consider initiating the integration process by drafting a Request for Comments (RFC). Refer to the concept of [RFC](https://pt.wikipedia.org/wiki/Request_for_Comments) for guidance. Emphasize the importance of maintaining clean and well-documented code to facilitate comprehension and collaboration among developers working on the integration.

Conduct a thorough evaluation of existing integrations within your team's codebase. Identify commonalities, potential areas for abstraction, and opportunities for optimization such as concurrency and batch processing. Advocate for **asynchronous and efficient communication**, providing practical implementation suggestions along with code examples.

Once your team aligns on the integration strategy, proceed to the coding phase.

## Mock the API

In my experience, achieving a **quality delivery is inseparable from rigorous testing**. Begin by creating test scenarios, especially when integrating with complex systems like ERPs.

Consider a hypothetical scenario where data from a list of companies within an ERP needs to be retrieved. As a personal recommendation, leverage tools like [MSW](https://github.com/mswjs/msw) for top-level mocks, which can significantly enhance the testing process.

Prioritizing testing not only ensures the reliability of your integration but also streamlines the development workflow, ultimately leading to a more robust and maintainable codebase.

So, firstly, setup your handlers:
```ts
const server = setupServer(
  // Describe network behavior with request handlers.
  // Tip: move the handlers into their own module and
  // import it across your browser and Node.js setups!
  http.get('/companies', ({ request, params, cookies }) => {
    return HttpResponse.json([
      {
        id: 'f8dd058f-9006-4174-8d49-e3086bc39c21',
        tradeName: `My Test Company`,
        employees: 10,
      },
      {
        id: '8ac96078-6434-4959-80ed-cc834e7fef61',
        tradeName: `My Test Company 2`,
        employees: 30,
      },
    ]);
  }),
);

// Enable request interception.
beforeAll(() => server.listen())

// Reset handlers so that each test could alter them
// without affecting other, unrelated tests.
afterEach(() => server.resetHandlers())

// Don't forget to clean up afterwards.
afterAll(() => server.close())
```

Then, create your test:
```ts
it('gets companies', async () => {
    // Your function to fetch the endpoint data (it can use fetch,
    // axios, etc).
    const companies = await getCompanies();

    expect(companies).toEqual([
      {
        id: 'f8dd058f-9006-4174-8d49-e3086bc39c21',
        tradeName: 'My Test Company',
        employees: 10,
      },
      {
        id: '8ac96078-6434-4959-80ed-cc834e7fef61',
        tradeName: 'My Test Company 2',
        employees: 30,
      },
    ]);
});
```

## Type the API
After creating the test cases, the next crucial step is to define the types for your API endpoints. I highly recommend leveraging [Zod](https://github.com/colinhacks/zod)💫 for its exceptional type safety and developer experience (DX).

An innovative approach is to **integrate GPT into this process**. Here's how:

![GPT Prompt](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dt469170ibqwxsduf2mn.png)

> Having integrated with endpoints featuring over 80 fields in the response, the time-saving potential of GPT becomes truly apparent.

Response:

![GPT response](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c0y6rdv0504is3es129j.png)

Wonderful, GPT saved us a lot of time, right?

One remarkable advantage of using Zod schemas is that they are **not strict**. In cases where certain fields, like `foo`, are not pertinent, Zod allows for their effortless removal. This flexibility ensures that your schema definitions remain concise and directly aligned with your specific needs.

```ts
// Zod schema without `foo`
export const getCompaniesResponseSchema = z.array(
  z.object({
    id: z.string().uuid(), // Improved schema definition
    tradeName: z.string(),
    employees: z.number().min(0), // A company cannot have less than 0 employees
  })
);

// Response type
export type GetCompaniesResponse = z.infer<typeof getCompaniesResponseSchema>
```
> Note that you can improve GPT responses with some [prompt engineering](https://en.wikipedia.org/wiki/Prompt_engineering) to add restrictions, descriptions, and possible meanings for each field, in the schema definition.
Feel free to explore.

Finally, use the schema to parse the responses:
```ts
/**
 * Gets companies list
 *
 * @example
 *   const companies = await getCompanies();
 *
 * @throws {ZodError} If parsing fails (response is not valid)
 */
const getCompanies = async (): Promise<GetCompaniesResponse> => {
  const response = await fetch('https://my-api.com/companies');

  return getCompaniesResponseSchema.parse(await response.json());
};
```

## Definitions of Done
As the integration process progresses, it's imperative to establish clear conditions for its conclusion. Ideally, these criteria should be aligned even before the initiation of RFCs, ensuring a cohesive and well-orchestrated development journey.

However, the final stage of delivery is the "Definitions of Done". It encapsulates all the necessary conditions for integration completion. These conditions encompass every integration requirement, essentially encapsulating all the data required for resolving the targeted problem.

Once these defined requirements are met, **the integration achieves its state of readiness**. These "Definitions of Done" serve as a comprehensive checklist, assuring that every aspect of the integration is not only functional but also aligned with the broader objectives it aims to address.

In essence, the clarity provided by these conclusive criteria ensures that the integration is not only technically sound but also effectively solves the targeted problem, contributing to a **successful and impactful project delivery**.
