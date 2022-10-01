
# dtrackr: Track your Data Pipelines <a href='https://dplyr.tidyverse.org'><img src='man/figures/logo.png' align="right" height="139" /></a>

<!-- badges: start -->

[![R-CMD-check](https://github.com/terminological/dtrackr/workflows/R-CMD-check/badge.svg)](https://github.com/terminological/dtrackr/actions)
[![DOI](https://zenodo.org/badge/335974323.svg)](https://zenodo.org/badge/latestdoi/335974323)
[![dtrackr status
badge](https://terminological.r-universe.dev/badges/dtrackr)](https://terminological.r-universe.dev)
<!-- badges: end -->

## Overview

Accurate documentation of a data pipeline is a first step to
reproducibility, and a flow chart describing the steps taken to prepare
data is a useful part of this documentation. In analyses that relies on
data that is frequently updated, documenting a data flow by copying and
pasting row counts into flowcharts in PowerPoint becomes quickly
tedious. With interactive data analysis, and particularly using
RMarkdown, code execution sometimes happens in a non-linear fashion, and
this can lead to, at best, confusion and at worst erroneous analysis.
Basing such documentation on what the code does when executed
sequentially can be inaccurate when the data has being analysed
interactively.

The goal of `dtrackr` is to take away this pain by instrumenting and
monitoring a dataframe through a `dplyr` pipeline, creating a
step-by-step summary of the important parts of the wrangling as it
actually happened to the dataframe, right into dataframe metadata
itself. This metadata can be used to generate documentation as a
flowchart, and allows both a quick overview of the data and also a
visual check of the actual data processing.

## Installation

In general use `dtrackr` is expected to be installed alongside the
`idyverse` set of packages. It is recommended to install `tidyverse`
first.

Binary packages of `dtrackr` are available on CRAN and r-universe for
`macOS` and `Windows`. `dtrackr` can be installed from source on Linux.
`dtrackr` has been tested on R versions 3.6, 4.0, 4.1 and 4.2.

You can install the released version of `dtrackr` from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("dtrackr")
```

### System dependencies for installation from source

For installation from source on in Linux, `dtrackr` has required
transitive dependencies on a few system libraries. These can be
installed with the following commands:

``` bash
# Ubuntu 20.04 and other debian based distributions:
sudo apt-get install libcurl4-openssl-dev libssl-dev librsvg2-dev \
  libicu-dev libnode-dev libpng-dev libjpeg-dev libpoppler-cpp-dev

# Centos 8
sudo dnf install libcurl-devel openssl-devel librsvg2-devel \
  libicu-devel libpng-devel libjpeg-turbo-devel poppler-devel

# for other linux distributions I suggest using the R pak library:
# install.packages("pak")
# pak::pkg_system_requirements("dtrackr")

# N.B. There are additional suggested R package dependencies on 
# the `tidyverse` and `rstudioapi` packages which have a longer set of dependencies. 
# We suggest you install them individually first if required.
```

### Alternative versions of `dtrackr`

Early release versions are available on the `r-universe`. This will
typically be more up to date than CRAN.

``` r
# Enable repository from terminological
options(repos = c(
  terminological = 'https://terminological.r-universe.dev',
  CRAN = 'https://cloud.r-project.org'))
# Download and install dtrackr in R
install.packages('dtrackr')
```

The unstable development version is available from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("terminological/dtrackr")
```

## Example usage

Suppose we are constructing a data set with out initial input being the
`iris` data. Our analysis depends on some `cutOff` parameter and we want
to prepare a stratified data set that excludes flowers with narrow
petals, and those with the biggest petals of each Species. With
`dtrackr` we can mix regular `dplyr` commands with additional `dtrackr`
commands such as `comment` and `status`, and an enhanced implementation
of `dplyr::filter`, called `exclude_all`, and `include_any`.

``` r
# a pipeline parameter
cutOff = 3

# the pipeline
dataset = iris %>% 
  track() %>%
  status() %>%
  group_by(Species) %>%
  status(
    short = p_count_if(Sepal.Width<cutOff), 
    long= p_count_if(Sepal.Width>=cutOff), 
    .messages=c("consisting of {short} short sepal <{cutOff}","and {long} long sepal >={cutOff}")
  )  %>%
  exclude_all(
    Petal.Width<0.3 ~ "excluding {.excluded} with narrow petals",
    Petal.Width == max(Petal.Width) ~ "and {.excluded} outlier"
  ) %>%
  comment("test message") %>%
  status(.messages = "{.count} of type {Species}") %>%
  ungroup() %>%
  status(.messages = "{.count} together with cutOff {cutOff}") 
```

Having prepared our dataset we conduct our analysis, and want to write
it up and prepare it for submission. As a key part of documenting the
data pipeline a visual summary is useful, and for bio-medical journals
or clinical trials often a requirement.

``` r

dataset %>% flowchart()
```

![](data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIKICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPgo8IS0tIEdlbmVyYXRlZCBieSBncmFwaHZpeiB2ZXJzaW9uIDIuNDAuMSAoMjAxNjEyMjUuMDMwNCkKIC0tPgo8IS0tIFRpdGxlOiAlMCBQYWdlczogMSAtLT4KPHN2ZyB3aWR0aD0iODA5cHQiIGhlaWdodD0iMzUwcHQiCiB2aWV3Qm94PSIwLjAwIDAuMDAgODA5LjQ3IDM1MC4wMCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayI+CjxnIGlkPSJncmFwaDAiIGNsYXNzPSJncmFwaCIgdHJhbnNmb3JtPSJzY2FsZSgxIDEpIHJvdGF0ZSgwKSB0cmFuc2xhdGUoNCAzNDYpIj4KPHRpdGxlPiUwPC90aXRsZT4KPHBvbHlnb24gZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSJ0cmFuc3BhcmVudCIgcG9pbnRzPSItNCw0IC00LC0zNDYgODA1LjQ3MTIsLTM0NiA4MDUuNDcxMiw0IC00LDQiLz4KPCEtLSAxNiYjNDU7Jmd0OzE3IC0tPgo8ZyBpZD0iZWRnZTEiIGNsYXNzPSJlZGdlIj4KPHRpdGxlPjE2OnMmIzQ1OyZndDsxNzwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgZD0iTTMzMi45MDYsLTQxLjRDMzMyLjkwNiwtNDEuNCAzMzIuOTA2LC0yOC40OTg3IDMzMi45MDYsLTI4LjQ5ODciLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjMzNC42NTYxLC0yOC40OTg3IDMzMi45MDYsLTIzLjQ5ODcgMzMxLjE1NjEsLTI4LjQ5ODcgMzM0LjY1NjEsLTI4LjQ5ODciLz4KPC9nPgo8IS0tIDEzJiM0NTsmZ3Q7MTYgLS0+CjxnIGlkPSJlZGdlNCIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+MTM6cyYjNDU7Jmd0OzE2PC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNNTguOTA2LC04My4yQzU4LjkwNiwtODMuMiA1OC45MDYsLTUzIDU4LjkwNiwtNTMgNTguOTA2LC01MyAzMDMuNTkwMywtNTMgMzAzLjU5MDMsLTUzIi8+Cjxwb2x5Z29uIGZpbGw9IiMwMDAwMDAiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSIzMDMuNTkwNCwtNTQuNzUwMSAzMDguNTkwMywtNTMgMzAzLjU5MDMsLTUxLjI1MDEgMzAzLjU5MDQsLTU0Ljc1MDEiLz4KPC9nPgo8IS0tIDE0JiM0NTsmZ3Q7MTYgLS0+CjxnIGlkPSJlZGdlMyIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+MTQ6cyYjNDU7Jmd0OzE2PC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNMzMyLjkwNiwtODMuMkMzMzIuOTA2LC04My4yIDMzMi45MDYsLTcwLjE3NzcgMzMyLjkwNiwtNzAuMTc3NyIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMzM0LjY1NjEsLTcwLjE3NzcgMzMyLjkwNiwtNjUuMTc3NyAzMzEuMTU2MSwtNzAuMTc3OCAzMzQuNjU2MSwtNzAuMTc3NyIvPgo8L2c+CjwhLS0gMTUmIzQ1OyZndDsxNiAtLT4KPGcgaWQ9ImVkZ2UyIiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT4xNTpzJiM0NTsmZ3Q7MTY8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAwMDAiIGQ9Ik02MDQuOTA2LC04My4yQzYwNC45MDYsLTgzLjIgNjA0LjkwNiwtNTMgNjA0LjkwNiwtNTMgNjA0LjkwNiwtNTMgMzYyLjUyMzksLTUzIDM2Mi41MjM5LC01MyIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMzYyLjUyNCwtNTEuMjUwMSAzNTcuNTIzOSwtNTMgMzYyLjUyMzksLTU0Ljc1MDEgMzYyLjUyNCwtNTEuMjUwMSIvPgo8L2c+CjwhLS0gMTAmIzQ1OyZndDsxMyAtLT4KPGcgaWQ9ImVkZ2U3IiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT4xMDpzJiM0NTsmZ3Q7MTM8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAwMDAiIGQ9Ik01OC45MDYsLTEzMS4yQzU4LjkwNiwtMTMxLjIgNTguOTA2LC0xMTguNTQwNyA1OC45MDYsLTExOC41NDA3Ii8+Cjxwb2x5Z29uIGZpbGw9IiMwMDAwMDAiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSI2MC42NTYxLC0xMTguNTQwNyA1OC45MDYsLTExMy41NDA3IDU3LjE1NjEsLTExOC41NDA3IDYwLjY1NjEsLTExOC41NDA3Ii8+CjwvZz4KPCEtLSAxMSYjNDU7Jmd0OzE0IC0tPgo8ZyBpZD0iZWRnZTYiIGNsYXNzPSJlZGdlIj4KPHRpdGxlPjExOnMmIzQ1OyZndDsxNDwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgZD0iTTMzMi45MDYsLTEzMS4yQzMzMi45MDYsLTEzMS4yIDMzMi45MDYsLTExOC41NDA3IDMzMi45MDYsLTExOC41NDA3Ii8+Cjxwb2x5Z29uIGZpbGw9IiMwMDAwMDAiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSIzMzQuNjU2MSwtMTE4LjU0MDcgMzMyLjkwNiwtMTEzLjU0MDcgMzMxLjE1NjEsLTExOC41NDA3IDMzNC42NTYxLC0xMTguNTQwNyIvPgo8L2c+CjwhLS0gMTImIzQ1OyZndDsxNSAtLT4KPGcgaWQ9ImVkZ2U1IiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT4xMjpzJiM0NTsmZ3Q7MTU8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAwMDAiIGQ9Ik02MDQuOTA2LC0xMzEuMkM2MDQuOTA2LC0xMzEuMiA2MDQuOTA2LC0xMTguNTQwNyA2MDQuOTA2LC0xMTguNTQwNyIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iNjA2LjY1NjEsLTExOC41NDA3IDYwNC45MDYsLTExMy41NDA3IDYwMy4xNTYxLC0xMTguNTQwNyA2MDYuNjU2MSwtMTE4LjU0MDciLz4KPC9nPgo8IS0tIDQmIzQ1OyZndDsxMCAtLT4KPGcgaWQ9ImVkZ2UxMCIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+NDpzJiM0NTsmZ3Q7MTA8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAwMDAiIGQ9Ik01OC45MDYsLTE3OS4yQzU4LjkwNiwtMTc5LjIgNTguOTA2LC0xNjYuNTQwNyA1OC45MDYsLTE2Ni41NDA3Ii8+Cjxwb2x5Z29uIGZpbGw9IiMwMDAwMDAiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSI2MC42NTYxLC0xNjYuNTQwNyA1OC45MDYsLTE2MS41NDA3IDU3LjE1NjEsLTE2Ni41NDA3IDYwLjY1NjEsLTE2Ni41NDA3Ii8+CjwvZz4KPCEtLSA0JiM0NTsmZ3Q7NyAtLT4KPGcgaWQ9ImVkZ2UxMyIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+NDplJiM0NTsmZ3Q7NzwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgZD0iTTExNy45MDYsLTE5OEMxMTcuOTA2LC0xOTggMTI3LjAyOSwtMTk4IDEyNy4wMjksLTE5OCIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMTI3LjAyOSwtMTk5Ljc1MDEgMTMyLjAyOSwtMTk4IDEyNy4wMjksLTE5Ni4yNTAxIDEyNy4wMjksLTE5OS43NTAxIi8+CjwvZz4KPCEtLSA1JiM0NTsmZ3Q7MTEgLS0+CjxnIGlkPSJlZGdlOSIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+NTpzJiM0NTsmZ3Q7MTE8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAwMDAiIGQ9Ik0zMzIuOTA2LC0xNzkuMkMzMzIuOTA2LC0xNzkuMiAzMzIuOTA2LC0xNjYuNTQwNyAzMzIuOTA2LC0xNjYuNTQwNyIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMzM0LjY1NjEsLTE2Ni41NDA3IDMzMi45MDYsLTE2MS41NDA3IDMzMS4xNTYxLC0xNjYuNTQwNyAzMzQuNjU2MSwtMTY2LjU0MDciLz4KPC9nPgo8IS0tIDUmIzQ1OyZndDs4IC0tPgo8ZyBpZD0iZWRnZTEyIiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT41OmUmIzQ1OyZndDs4PC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNMzkzLjkwNiwtMTk4QzM5My45MDYsLTE5OCA0MDMuMDEyOSwtMTk4IDQwMy4wMTI5LC0xOTgiLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjQwMy4wMTMsLTE5OS43NTAxIDQwOC4wMTI5LC0xOTggNDAzLjAxMjksLTE5Ni4yNTAxIDQwMy4wMTMsLTE5OS43NTAxIi8+CjwvZz4KPCEtLSA2JiM0NTsmZ3Q7MTIgLS0+CjxnIGlkPSJlZGdlOCIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+NjpzJiM0NTsmZ3Q7MTI8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAwMDAiIGQ9Ik02MDQuOTA2LC0xNzkuMkM2MDQuOTA2LC0xNzkuMiA2MDQuOTA2LC0xNjYuNTQwNyA2MDQuOTA2LC0xNjYuNTQwNyIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iNjA2LjY1NjEsLTE2Ni41NDA3IDYwNC45MDYsLTE2MS41NDA3IDYwMy4xNTYxLC0xNjYuNTQwNyA2MDYuNjU2MSwtMTY2LjU0MDciLz4KPC9nPgo8IS0tIDYmIzQ1OyZndDs5IC0tPgo8ZyBpZD0iZWRnZTExIiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT42OmUmIzQ1OyZndDs5PC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjMDAwMDAwIiBkPSJNNjY1LjkwNiwtMTk4QzY2NS45MDYsLTE5OCA2NzUuMDEyOSwtMTk4IDY3NS4wMTI5LC0xOTgiLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjY3NS4wMTMsLTE5OS43NTAxIDY4MC4wMTI5LC0xOTggNjc1LjAxMjksLTE5Ni4yNTAxIDY3NS4wMTMsLTE5OS43NTAxIi8+CjwvZz4KPCEtLSAzJiM0NTsmZ3Q7NCAtLT4KPGcgaWQ9ImVkZ2UxNiIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+MzpzJiM0NTsmZ3Q7NDwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgZD0iTTMzMi45MDYsLTI0N0MzMzIuOTA2LC0yNDcgNTguOTA2LC0yNDcgNTguOTA2LC0yNDcgNTguOTA2LC0yNDcgNTguOTA2LC0yMjIuMjA3MSA1OC45MDYsLTIyMi4yMDcxIi8+Cjxwb2x5Z29uIGZpbGw9IiMwMDAwMDAiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSI2MC42NTYxLC0yMjIuMjA3MSA1OC45MDYsLTIxNy4yMDcxIDU3LjE1NjEsLTIyMi4yMDcxIDYwLjY1NjEsLTIyMi4yMDcxIi8+CjwvZz4KPCEtLSAzJiM0NTsmZ3Q7NSAtLT4KPGcgaWQ9ImVkZ2UxNSIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+MzpzJiM0NTsmZ3Q7NTwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgZD0iTTMzMi45MDYsLTIzNUMzMzIuOTA2LC0yMzUgMzMyLjkwNiwtMjIyLjQ2MjIgMzMyLjkwNiwtMjIyLjQ2MjIiLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjMzNC42NTYxLC0yMjIuNDYyMiAzMzIuOTA2LC0yMTcuNDYyMiAzMzEuMTU2MSwtMjIyLjQ2MjMgMzM0LjY1NjEsLTIyMi40NjIyIi8+CjwvZz4KPCEtLSAzJiM0NTsmZ3Q7NiAtLT4KPGcgaWQ9ImVkZ2UxNCIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+MzpzJiM0NTsmZ3Q7NjwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgZD0iTTMzMi45MDYsLTI0N0MzMzIuOTA2LC0yNDcgNjA0LjkwNiwtMjQ3IDYwNC45MDYsLTI0NyA2MDQuOTA2LC0yNDcgNjA0LjkwNiwtMjIyLjIwNzEgNjA0LjkwNiwtMjIyLjIwNzEiLz4KPHBvbHlnb24gZmlsbD0iIzAwMDAwMCIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjYwNi42NTYxLC0yMjIuMjA3MSA2MDQuOTA2LC0yMTcuMjA3MSA2MDMuMTU2MSwtMjIyLjIwNzEgNjA2LjY1NjEsLTIyMi4yMDcxIi8+CjwvZz4KPCEtLSAyJiM0NTsmZ3Q7MyAtLT4KPGcgaWQ9ImVkZ2UxNyIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+MjpzJiM0NTsmZ3Q7MzwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzAwMDAwMCIgZD0iTTMzMi45MDYsLTI3Ni42QzMzMi45MDYsLTI3Ni42IDMzMi45MDYsLTI2My42OTg3IDMzMi45MDYsLTI2My42OTg3Ii8+Cjxwb2x5Z29uIGZpbGw9IiMwMDAwMDAiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSIzMzQuNjU2MSwtMjYzLjY5ODcgMzMyLjkwNiwtMjU4LjY5ODcgMzMxLjE1NjEsLTI2My42OTg3IDMzNC42NTYxLC0yNjMuNjk4NyIvPgo8L2c+CjwhLS0gMSYjNDU7Jmd0OzIgLS0+CjxnIGlkPSJlZGdlMTgiIGNsYXNzPSJlZGdlIj4KPHRpdGxlPjE6cyYjNDU7Jmd0OzI8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAwMDAiIGQ9Ik0zMzIuOTA2LC0zMTguMkMzMzIuOTA2LC0zMTguMiAzMzIuOTA2LC0zMDUuMjk4NyAzMzIuOTA2LC0zMDUuMjk4NyIvPgo8cG9seWdvbiBmaWxsPSIjMDAwMDAwIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMzM0LjY1NjEsLTMwNS4yOTg3IDMzMi45MDYsLTMwMC4yOTg3IDMzMS4xNTYxLC0zMDUuMjk4NyAzMzQuNjU2MSwtMzA1LjI5ODciLz4KPC9nPgo8IS0tIDE3IC0tPgo8ZyBpZD0ibm9kZTEiIGNsYXNzPSJub2RlIj4KPHRpdGxlPjE3PC90aXRsZT4KPHBvbHlnb24gZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjM4NS40OTQ4LC0yMy40MDMzIDI4MC4zMTcyLC0yMy40MDMzIDI4MC4zMTcyLC0uMTk2NyAzODUuNDk0OCwtLjE5NjcgMzg1LjQ5NDgsLTIzLjQwMzMiLz4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSIyODcuMTEyIiB5PSItOS40IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+MTExIHRvZ2V0aGVyIHdpdGggY3V0T2ZmIDM8L3RleHQ+CjwvZz4KPCEtLSAxNiAtLT4KPGcgaWQ9Im5vZGUyIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT4xNjwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSIzNTcuMDgwNywtNjUuMDAzMyAzMDguNzMxMywtNjUuMDAzMyAzMDguNzMxMywtNDEuNzk2NyAzNTcuMDgwNywtNDEuNzk2NyAzNTcuMDgwNywtNjUuMDAzMyIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjMxNS41NjkyIiB5PSItNTEiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj4xMTEgaXRlbXM8L3RleHQ+CjwvZz4KPCEtLSAxMyAtLT4KPGcgaWQ9Im5vZGUzIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT4xMzwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSI5Ni4zMjI4LC0xMTMuMiAyMS40ODkyLC0xMTMuMiAyMS40ODkyLC04My4yIDk2LjMyMjgsLTgzLjIgOTYuMzIyOCwtMTEzLjIiLz4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSIyOC40NDc2IiB5PSItMTAwIiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtd2VpZ2h0PSJib2xkIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPlNwZWNpZXM6c2V0b3NhPC90ZXh0Pgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjI4LjQ0NzYiIHk9Ii05MiIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPjE1IG9mIHR5cGUgc2V0b3NhPC90ZXh0Pgo8L2c+CjwhLS0gMTQgLS0+CjxnIGlkPSJub2RlNCIgY2xhc3M9Im5vZGUiPgo8dGl0bGU+MTQ8L3RpdGxlPgo8cG9seWdvbiBmaWxsPSIjZmZmZmZmIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMzc1LjQ3NzUsLTExMy4yIDI5MC4zMzQ1LC0xMTMuMiAyOTAuMzM0NSwtODMuMiAzNzUuNDc3NSwtODMuMiAzNzUuNDc3NSwtMTEzLjIiLz4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSIyOTcuMTIwOCIgeT0iLTEwMCIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXdlaWdodD0iYm9sZCIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj5TcGVjaWVzOnZlcnNpY29sb3I8L3RleHQ+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iMjk3LjEyMDgiIHk9Ii05MiIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPjQ5IG9mIHR5cGUgdmVyc2ljb2xvcjwvdGV4dD4KPC9nPgo8IS0tIDE1IC0tPgo8ZyBpZD0ibm9kZTUiIGNsYXNzPSJub2RlIj4KPHRpdGxlPjE1PC90aXRsZT4KPHBvbHlnb24gZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjY0NS4wODksLTExMy4yIDU2NC43MjMsLTExMy4yIDU2NC43MjMsLTgzLjIgNjQ1LjA4OSwtODMuMiA2NDUuMDg5LC0xMTMuMiIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjU3MS41NjQ4IiB5PSItMTAwIiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtd2VpZ2h0PSJib2xkIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPlNwZWNpZXM6dmlyZ2luaWNhPC90ZXh0Pgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjU3MS41NjQ4IiB5PSItOTIiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj40NyBvZiB0eXBlIHZpcmdpbmljYTwvdGV4dD4KPC9nPgo8IS0tIDEwIC0tPgo8ZyBpZD0ibm9kZTYiIGNsYXNzPSJub2RlIj4KPHRpdGxlPjEwPC90aXRsZT4KPHBvbHlnb24gZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjkzLjE0NjQsLTE2MS4yIDI0LjY2NTYsLTE2MS4yIDI0LjY2NTYsLTEzMS4yIDkzLjE0NjQsLTEzMS4yIDkzLjE0NjQsLTE2MS4yIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iMzEuNzg2IiB5PSItMTQ4IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtd2VpZ2h0PSJib2xkIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPlNwZWNpZXM6c2V0b3NhPC90ZXh0Pgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjMxLjc4NiIgeT0iLTE0MCIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPnRlc3QgbWVzc2FnZTwvdGV4dD4KPC9nPgo8IS0tIDExIC0tPgo8ZyBpZD0ibm9kZTciIGNsYXNzPSJub2RlIj4KPHRpdGxlPjExPC90aXRsZT4KPHBvbHlnb24gZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjM3Mi4yOTk3LC0xNjEuMiAyOTMuNTEyMywtMTYxLjIgMjkzLjUxMjMsLTEzMS4yIDM3Mi4yOTk3LC0xMzEuMiAzNzIuMjk5NywtMTYxLjIiLz4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSIzMDAuNDU5MiIgeT0iLTE0OCIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXdlaWdodD0iYm9sZCIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj5TcGVjaWVzOnZlcnNpY29sb3I8L3RleHQ+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iMzAwLjQ1OTIiIHk9Ii0xNDAiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj50ZXN0IG1lc3NhZ2U8L3RleHQ+CjwvZz4KPCEtLSAxMiAtLT4KPGcgaWQ9Im5vZGU4IiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT4xMjwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSI2NDEuOTExNiwtMTYxLjIgNTY3LjkwMDQsLTE2MS4yIDU2Ny45MDA0LC0xMzEuMiA2NDEuOTExNiwtMTMxLjIgNjQxLjkxMTYsLTE2MS4yIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iNTc0LjkwMzIiIHk9Ii0xNDgiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC13ZWlnaHQ9ImJvbGQiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+U3BlY2llczp2aXJnaW5pY2E8L3RleHQ+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iNTc0LjkwMzIiIHk9Ii0xNDAiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj50ZXN0IG1lc3NhZ2U8L3RleHQ+CjwvZz4KPCEtLSA0IC0tPgo8ZyBpZD0ibm9kZTkiIGNsYXNzPSJub2RlIj4KPHRpdGxlPjQ8L3RpdGxlPgo8cG9seWdvbiBmaWxsPSIjZmZmZmZmIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMTE3LjcxODEsLTIxNy4yIC4wOTM5LC0yMTcuMiAuMDkzOSwtMTc5LjIgMTE3LjcxODEsLTE3OS4yIDExNy43MTgxLC0yMTcuMiIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjciIHk9Ii0yMDQiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC13ZWlnaHQ9ImJvbGQiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+U3BlY2llczpzZXRvc2E8L3RleHQ+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iNyIgeT0iLTE5NiIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPmNvbnNpc3Rpbmcgb2YgMiBzaG9ydCBzZXBhbCAmbHQ7MzwvdGV4dD4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSI3IiB5PSItMTg4IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+YW5kIDQ4IGxvbmcgc2VwYWwgJmd0Oz0zPC90ZXh0Pgo8L2c+CjwhLS0gNSAtLT4KPGcgaWQ9Im5vZGUxMCIgY2xhc3M9Im5vZGUiPgo8dGl0bGU+NTwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSIzOTQuMTY1NSwtMjE3LjIgMjcxLjY0NjUsLTIxNy4yIDI3MS42NDY1LC0xNzkuMiAzOTQuMTY1NSwtMTc5LjIgMzk0LjE2NTUsLTIxNy4yIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iMjc4Ljc3NjQiIHk9Ii0yMDQiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC13ZWlnaHQ9ImJvbGQiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+U3BlY2llczp2ZXJzaWNvbG9yPC90ZXh0Pgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjI3OC43NzY0IiB5PSItMTk2IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+Y29uc2lzdGluZyBvZiAzNCBzaG9ydCBzZXBhbCAmbHQ7MzwvdGV4dD4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSIyNzguNzc2NCIgeT0iLTE4OCIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPmFuZCAxNiBsb25nIHNlcGFsICZndDs9MzwvdGV4dD4KPC9nPgo8IS0tIDYgLS0+CjxnIGlkPSJub2RlMTEiIGNsYXNzPSJub2RlIj4KPHRpdGxlPjY8L3RpdGxlPgo8cG9seWdvbiBmaWxsPSIjZmZmZmZmIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iNjY2LjE2NTUsLTIxNy4yIDU0My42NDY1LC0yMTcuMiA1NDMuNjQ2NSwtMTc5LjIgNjY2LjE2NTUsLTE3OS4yIDY2Ni4xNjU1LC0yMTcuMiIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjU1MC43NzY0IiB5PSItMjA0IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtd2VpZ2h0PSJib2xkIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPlNwZWNpZXM6dmlyZ2luaWNhPC90ZXh0Pgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjU1MC43NzY0IiB5PSItMTk2IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+Y29uc2lzdGluZyBvZiAyMSBzaG9ydCBzZXBhbCAmbHQ7MzwvdGV4dD4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSI1NTAuNzc2NCIgeT0iLTE4OCIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPmFuZCAyOSBsb25nIHNlcGFsICZndDs9MzwvdGV4dD4KPC9nPgo8IS0tIDcgLS0+CjxnIGlkPSJub2RlMTIiIGNsYXNzPSJub2RlIj4KPHRpdGxlPjc8L3RpdGxlPgo8cG9seWdvbiBmaWxsPSIjY2NjY2NjIiBzdHJva2U9IiMwMDAwMDAiIHBvaW50cz0iMjU3LjQ4NDMsLTIxNy4yIDEzMi4zMjc3LC0yMTcuMiAxMzIuMzI3NywtMTc5LjIgMjU3LjQ4NDMsLTE3OS4yIDI1Ny40ODQzLC0yMTcuMiIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjEzOS4xMTcyIiB5PSItMjA0IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtd2VpZ2h0PSJib2xkIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPlNwZWNpZXM6c2V0b3NhPC90ZXh0Pgo8dGV4dCB0ZXh0LWFuY2hvcj0ic3RhcnQiIHg9IjEzOS4xMTcyIiB5PSItMTk2IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+ZXhjbHVkaW5nIDM0IHdpdGggbmFycm93IHBldGFsczwvdGV4dD4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSIxMzkuMTE3MiIgeT0iLTE4OCIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPmFuZCAxIG91dGxpZXI8L3RleHQ+CjwvZz4KPCEtLSA4IC0tPgo8ZyBpZD0ibm9kZTEzIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT44PC90aXRsZT4KPHBvbHlnb24gZmlsbD0iI2NjY2NjYyIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjUyOS41MzY1LC0yMTcuMiA0MDguMjc1NSwtMjE3LjIgNDA4LjI3NTUsLTE3OS4yIDUyOS41MzY1LC0xNzkuMiA1MjkuNTM2NSwtMjE3LjIiLz4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSI0MTUuMzQwOCIgeT0iLTIwNCIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXdlaWdodD0iYm9sZCIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj5TcGVjaWVzOnZlcnNpY29sb3I8L3RleHQ+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iNDE1LjM0MDgiIHk9Ii0xOTYiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj5leGNsdWRpbmcgMCB3aXRoIG5hcnJvdyBwZXRhbHM8L3RleHQ+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iNDE1LjM0MDgiIHk9Ii0xODgiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj5hbmQgMSBvdXRsaWVyPC90ZXh0Pgo8L2c+CjwhLS0gOSAtLT4KPGcgaWQ9Im5vZGUxNCIgY2xhc3M9Im5vZGUiPgo8dGl0bGU+OTwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IiNjY2NjY2MiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSI4MDEuNTM2NSwtMjE3LjIgNjgwLjI3NTUsLTIxNy4yIDY4MC4yNzU1LC0xNzkuMiA4MDEuNTM2NSwtMTc5LjIgODAxLjUzNjUsLTIxNy4yIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iNjg3LjM0MDgiIHk9Ii0yMDQiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC13ZWlnaHQ9ImJvbGQiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+U3BlY2llczp2aXJnaW5pY2E8L3RleHQ+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iNjg3LjM0MDgiIHk9Ii0xOTYiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj5leGNsdWRpbmcgMCB3aXRoIG5hcnJvdyBwZXRhbHM8L3RleHQ+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iNjg3LjM0MDgiIHk9Ii0xODgiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSI4LjAwIiBmaWxsPSIjMDAwMDAwIj5hbmQgMyBvdXRsaWVyPC90ZXh0Pgo8L2c+CjwhLS0gMyAtLT4KPGcgaWQ9Im5vZGUxNSIgY2xhc3M9Im5vZGUiPgo8dGl0bGU+MzwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSIzNzIuMzA2OSwtMjU4LjYwMzMgMjkzLjUwNTEsLTI1OC42MDMzIDI5My41MDUxLC0yMzUuMzk2NyAzNzIuMzA2OSwtMjM1LjM5NjcgMzcyLjMwNjksLTI1OC42MDMzIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iMzAwLjQ1NTYiIHk9Ii0yNDQuNiIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPnN0cmF0aWZ5IGJ5IFNwZWNpZXM8L3RleHQ+CjwvZz4KPCEtLSAyIC0tPgo8ZyBpZD0ibm9kZTE2IiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT4yPC90aXRsZT4KPHBvbHlnb24gZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjMDAwMDAwIiBwb2ludHM9IjM1Ny4wODA3LC0zMDAuMjAzMyAzMDguNzMxMywtMzAwLjIwMzMgMzA4LjczMTMsLTI3Ni45OTY3IDM1Ny4wODA3LC0yNzYuOTk2NyAzNTcuMDgwNywtMzAwLjIwMzMiLz4KPHRleHQgdGV4dC1hbmNob3I9InN0YXJ0IiB4PSIzMTUuNTY5MiIgeT0iLTI4Ni4yIiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtc2l6ZT0iOC4wMCIgZmlsbD0iIzAwMDAwMCI+MTUwIGl0ZW1zPC90ZXh0Pgo8L2c+CjwhLS0gMSAtLT4KPGcgaWQ9Im5vZGUxNyIgY2xhc3M9Im5vZGUiPgo8dGl0bGU+MTwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzAwMDAwMCIgcG9pbnRzPSIzNTcuMDgwNywtMzQxLjgwMzMgMzA4LjczMTMsLTM0MS44MDMzIDMwOC43MzEzLC0zMTguNTk2NyAzNTcuMDgwNywtMzE4LjU5NjcgMzU3LjA4MDcsLTM0MS44MDMzIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJzdGFydCIgeD0iMzE1LjU2OTIiIHk9Ii0zMjcuOCIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjguMDAiIGZpbGw9IiMwMDAwMDAiPjE1MCBpdGVtczwvdGV4dD4KPC9nPgo8L2c+Cjwvc3ZnPgo=)

And your publication ready data pipeline, with any assumptions you care
to document, is creates in a format of your choice (as long as that
choice is one of `pdf`, `png`, `svg` or `ps`), ready for submission to
Nature.

This is a trivial example, but the more complex the pipeline, the bigger
benefit you will get.

Check out the [main documentation for detailed
examples](https://terminological.github.io/dtrackr/)

## Testing and integration

For testing `dtrackr` uses the `testthat` framework. It is configured to
run both the unit tests and the functional tests in the code examples.

``` r
# assuming dtrackr has been cloned from github into the working directory 
# location

devtools::load_all()

# Long list of system dependencies in Ubuntu 20.04 including all suggested 
# dependencies:
# librsvg2-dev libicu-dev libcurl4-openssl-dev libssl-dev libnode-dev make 
# pandoc imagemagick libmagick++-dev gsfonts default-jdk libxml2-dev 
# zlib1g-dev libfontconfig1-dev libfreetype6-dev libfribidi-dev libharfbuzz-dev 
# libjpeg-dev libpng-dev libtiff-dev git libgit2-dev

pak::::local_system_requirements("ubuntu","20.04")
install.packages(c("here","tidyverse","devtools","testthat","pkgdown"))

# Examples:
devtools::run_examples()

# automated testing with testthat (also runs all examples):
devtools::test()

# pkgdown site building (which executes all the vignettes):
pkgdown::build_site()
```

Github workflows are enabled on this repository for continuous
integration. These perform an `R CMD check` on code commits, which will
run all `testthat` unit tests, run all man page examples, and build all
the vignettes.

For vignette building there are dependencies on the `tidyverse` package,
as well as other system libraries required for vignette building with
`pandoc`. The CI tests check the library can be installed on macOs,
windows, and Ubuntu, with R versions 3.6.1, 4.1, 4.2 and the R
development branch.

On tagged releases, additional Github workflows are triggered by in the
`r-universe` repository, to build binary releases on a range of
platforms.
