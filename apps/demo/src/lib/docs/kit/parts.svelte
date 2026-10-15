<!--
  @component
  A row of `Part`s, linked by `Arrow`s where one depends on another, ruled as one grid that fills
  its example's width: the parts share their lines, each arrow fills a narrow hatched cell between
  the two parts it links, and the grid's outer corners carry the site's crosses. Every part gets
  the same width and height, and what a part shows fills it, so values, results and `actions` line
  up across the row. Two parts side by side with no arrow between them don't depend on each other;
  on a narrow screen the second one starts a new row. A chain of three or more linked parts is too
  wide for a phone, so there it stacks, with its arrows pointing down. Set `stack` to stack a
  shorter chain too, when its parts hold more than a phone's half width can show (a sentence, a
  component).

  ```svelte
  <Parts>
    <Part code label="countAtom">…</Part>
    <Arrow label="get" pulse={count.current} />
    <Part code label="doubledAtom">…</Part>
    <Part code label="greetingAtom">…</Part>
  </Parts>
  ```

  `columns` lays the parts out as a grid of that many columns instead (no arrows), such as four
  hooks on the same atom in two rows of two; on a narrow screen it is one column.

  Other attributes, and classes (`class="mt-4"`), go on the grid.
-->
<script lang="ts">
  import type { Snippet } from "svelte";
  import type { HTMLAttributes } from "svelte/elements";

  interface Props extends HTMLAttributes<HTMLDivElement> {
    readonly children: Snippet;
    /** Lays the parts out in this many columns, for a grid rather than a row. */
    readonly columns?: number;
    /** Stacks the parts on a narrow screen, however few there are. */
    readonly stack?: boolean;
  }

  const {
    children,
    class: className,
    columns,
    stack = false,
    ...rest
  }: Props = $props();
</script>

<div class={["parts not-prose", className]} {...rest}>
  <div
    class={["row", stack && "stack", columns !== undefined && "grid"]}
    style:--columns={columns}
  >
    {@render children()}
  </div>
  <span aria-hidden="true" class="corner tl"></span>
  <span aria-hidden="true" class="corner tr"></span>
  <span aria-hidden="true" class="corner bl"></span>
  <span aria-hidden="true" class="corner br"></span>
</div>

<style>
  .parts {
    container-type: inline-size;
    position: relative;
  }
  /* Each cell overlaps the next by its 1px border, so neighbours share one line; the row's padding
     gives the last ones back the pixel. */
  .row {
    align-items: stretch;
    display: flex;
    flex-wrap: wrap;
    padding: 0 1px 1px 0;
  }
  .row > :global(*) {
    margin: 0 -1px -1px 0;
  }
  .row.grid {
    display: grid;
    grid-template-columns: repeat(var(--columns), minmax(0, 1fr));
  }
  .parts .row > :global(.part) {
    flex: 1 1 0;
    min-width: 0;
  }
  /* The grid draws the crosses at its own corners; the parts' would mark every inner joint. */
  .parts .row > :global(.part > .corner) {
    display: none;
  }
  /* What a part shows fills its width and starts at its top, so values line up across the row. */
  .parts .row > :global(.part > .body) {
    align-content: start;
  }
  .parts .row > :global(.part > .body > *) {
    align-items: stretch;
    text-align: center;
    width: 100%;
  }
  /* An arrow is a narrow cell of its own, hatched like a drawing's dimension band. */
  .parts .row > :global(.arrow) {
    align-self: stretch;
    background: repeating-linear-gradient(
      135deg,
      transparent 0 6px,
      color-mix(in oklab, var(--border) 50%, transparent) 6px 7px
    );
    border: 1px solid var(--border);
    flex: none;
    padding: 0.5rem 0.75rem;
    width: 4.75rem;
  }
  .corner {
    height: 1px;
    pointer-events: none;
    position: absolute;
    width: 1px;
  }
  .corner::before,
  .corner::after {
    content: "";
    position: absolute;
  }
  .corner::before {
    border-top: 1px solid var(--cross);
    left: -5px;
    top: 0;
    width: 11px;
  }
  .corner::after {
    border-left: 1px solid var(--cross);
    height: 11px;
    left: 0;
    top: -5px;
  }
  .tl,
  .bl {
    left: 0;
  }
  .tr,
  .br {
    left: calc(100% - 1px);
  }
  .tl,
  .tr {
    top: 0;
  }
  .bl,
  .br {
    top: calc(100% - 1px);
  }
  /* Narrow: each group of linked parts gets its own row, and a grid is one column. */
  @container (width < 34rem) {
    .row.grid {
      grid-template-columns: minmax(0, 1fr);
    }
    .parts .row.stack,
    .parts .row:has(> :global(.part + .arrow + .part + .arrow + .part)) {
      flex-direction: column;
      & > :global(.part) {
        flex: none;
      }
      & > :global(.arrow) {
        flex-direction: row-reverse;
        gap: 0.6rem;
        min-height: 2.75rem;
        width: auto;
      }
      & > :global(.arrow .line) {
        align-self: center;
        rotate: 90deg;
        width: 1.75rem;
      }
    }
    .parts .row > :global(.part + .part) {
      flex-basis: 100%;
    }
  }
</style>
